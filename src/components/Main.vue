<template>
  <div class="main">
    <div class="input">
      <div class="left">
        <div class="description">アカウントA (例: @arkw@misskey.io)</div>
        <input type="text" class="text" v-model="inputModel[0]" />
      </div>
      <div class="right">
        <div class="description">アカウントB (例: @arkw@mi.arkw.work)</div>
        <input type="text" class="text" v-model="inputModel[1]" />
      </div>
      <div class="update"><div class="button" @click="update">更新</div></div>
    </div>
    <div class="mode">
      <select v-model="mode" @change="changeList">
        <option value="">全てのアカウント</option>
        <option value="or">ORで絞り込む</option>
        <option value="and">ANDで絞り込む</option>
      </select>
    </div>
    <div class="condition">
      <Checkbox v-model:value="isShow[0].following" @change="changeList" label="Aフォロー" />
      <Checkbox v-model:value="isShow[0].followers" @change="changeList" label="Aフォロワー" />
      <Checkbox v-model:value="isShow[1].following" @change="changeList" label="Bフォロー" />
      <Checkbox v-model:value="isShow[1].followers" @change="changeList" label="Bフォロワー" />
    </div>
    <div class="onetouch">
      <div class="description">ワンタッチ:</div>
      <div class="button" @click="changeCondition(0)">A片思い</div>
      <div class="button" @click="changeCondition(1)">A片思われ</div>
      <div class="button" @click="changeCondition(2)">B片思い</div>
      <div class="button" @click="changeCondition(3)">B片思われ</div>
      <div class="button" @click="changeCondition(4)">Aだけ未フォロー</div>
      <div class="button" @click="changeCondition(5)">Bだけ未フォロー</div>
    </div>
    <div class="list">
      <div v-for="note in listShow" v-bind:key="note.id">
        <Note :avatarUrl="note.avatarUrl" :name="note.name" :userid="note.id" :value="note.value" />
      </div>
    </div>
  </div>
</template>

<script setup lang="js">
import { reactive, ref } from 'vue'
import axios from 'axios'
import Checkbox from './Checkbox.vue'
import Note from './Note.vue'
const inputModel = ref([''], [''])
const list = new Array()
const listShow = ref(new Array())
const mode = ref('')
const isShow = reactive([
  {
    followers: true,
    following: true,
  },
  {
    followers: true,
    following: true,
  },
])

const getList = async (url, id, max, type) => {
  const output = new Array()
  const set = new Set()
  let count = 0
  let next = null
  for (let i = 0; i < max / 100 + 2; i++) {
    let response
    if (next != null) {
      response = await axios.post(url, {
        userId: id,
        limit: 100,
        untilId: next,
      })
    } else {
      response = await axios.post(url, {
        userId: id,
        limit: 100,
      })
    }
    for (let j = 0; j < response.data.length; j++) {
      if (j == response.data.length - 1) {
        next = response.data[j].id
      } else {
        if (type == 'followee') {
          const id = '@' + response.data[j].followee.username + '@' + response.data[j].followee.host
          if (set.has(id) == false) {
            output.push({
              id: id,
              avatarUrl: response.data[j].followee.avatarUrl,
              name: response.data[j].followee.name,
            })
            set.add(id)
          }
        } else if (type == 'follower') {
          const id = '@' + response.data[j].follower.username + '@' + response.data[j].follower.host
          if (set.has(id) == false) {
            output.push({
              id: id,
              avatarUrl: response.data[j].follower.avatarUrl,
              name: response.data[j].follower.name,
            })
            set.add(id)
          }
        }
        count++
      }
    }
    if (count >= max) {
      break
    }
  }
  return output
}

const getUser = async (id) => {
  const output = { followers: null, following: null }
  const userhost = id.split('@')
  const user = await axios.post(
    'https://' + userhost[2] + '/api/users/search-by-username-and-host',
    {
      username: userhost[1],
      host: userhost[2],
      limit: 1,
      detail: false,
    },
  )
  const stats = await axios.post('https://' + userhost[2] + '/api/users/show', {
    userId: user.data[0].id,
  })
  output.followers = await getList(
    'https://' + userhost[2] + '/api/users/followers',
    user.data[0].id,
    stats.data.followingCount,
    'follower',
  )
  output.following = await getList(
    'https://' + userhost[2] + '/api/users/following',
    user.data[0].id,
    stats.data.followingCount,
    'followee',
  )
  return output
}

const isValid = (str) => {
  if (typeof str !== 'string') return false
  if (!str.startsWith('@')) return false
  const matches = str.match(/@/g)
  return matches !== null && matches.length >= 2
}

const addList = (id, avatarUrl, name, index, type) => {
  let position = list.findIndex((el) => el.id === id)
  if (position == -1) {
    list.push({
      id: id,
      avatarUrl: avatarUrl,
      name: name,
      value: [
        {
          followers: false,
          following: false,
        },
        {
          followers: false,
          following: false,
        },
      ],
    })
    position = list.length - 1
  }
  if (type == 'followee') {
    list[position].value[index].following = true
  } else if (type == 'follower') {
    list[position].value[index].followers = true
  }
  return
}

const update = async () => {
  list.splice(0)
  for (let i = 0; i < 2; i++) {
    if (isValid(inputModel.value[i]) == true) {
      const data = await getUser(inputModel.value[i])
      for (let j = 0; j < data.following.length; j++) {
        addList(
          data.following[j].id,
          data.following[j].avatarUrl,
          data.following[j].name,
          i,
          'followee',
        )
      }
      for (let j = 0; j < data.followers.length; j++) {
        addList(
          data.followers[j].id,
          data.followers[j].avatarUrl,
          data.followers[j].name,
          i,
          'follower',
        )
      }
    }
  }
  changeList()
}

const changeList = () => {
  listShow.value = []
  if (mode.value == '') {
    isShow[0].followers = true
    isShow[0].following = true
    isShow[1].followers = true
    isShow[1].following = true
  }
  list.forEach((item) => {
    let add = false
    if (mode.value == 'or') {
      add =
        (isShow[0].followers && item.value[0].followers) ||
        (isShow[0].following && item.value[0].following) ||
        (isShow[1].followers && item.value[1].followers) ||
        (isShow[1].following && item.value[1].following)
    } else if (mode.value == 'and') {
      add =
        isShow[0].followers === item.value[0].followers &&
        isShow[0].following === item.value[0].following &&
        isShow[1].followers === item.value[1].followers &&
        isShow[1].following === item.value[1].following
    } else {
      add = true
    }
    if (add == true) {
      listShow.value.push({ ...item })
    }
  })
}

const changeCondition = (i) => {
  mode.value = 'and'
  switch (i) {
    case 0:
      isShow[0].followers = false
      isShow[0].following = true
      isShow[1].followers = false
      isShow[1].following = false
      break
    case 1:
      isShow[0].followers = true
      isShow[0].following = false
      isShow[1].followers = false
      isShow[1].following = false
      break
    case 2:
      isShow[0].followers = false
      isShow[0].following = false
      isShow[1].followers = false
      isShow[1].following = true
      break
    case 3:
      isShow[0].followers = false
      isShow[0].following = false
      isShow[1].followers = true
      isShow[1].following = false
      break
    case 4:
      isShow[0].followers = false
      isShow[0].following = false
      isShow[1].followers = false
      isShow[1].following = true
      break
    case 5:
      isShow[0].followers = false
      isShow[0].following = true
      isShow[1].followers = false
      isShow[1].following = false
      break
  }
}
</script>

<style scoped lang="css">
.main {
  padding: 20px;
}

.main .information {
  margin-bottom: 10px;
  padding: 10px;
  color: #757575;
  background-color: #b3e5fc;
  border-radius: 5px;
}

.main .input {
  overflow: hidden;
}

.main .left,
.main .right {
  width: 340px;
  float: left;
}

.main .input .description {
  height: 28px;
  color: #757575;
}

.main .input .text {
  width: 320px;
  height: 32px;
  margin-right: 10px;
  padding-left: 4px;
  float: left;
  border: 0;
  border-radius: 4px;
  outline: 0;
}

.main .input .text:focus {
  border: 1px solid #8bc34a;
  outline: 0;
}

.main .update .button {
  width: 100px;
  height: 32px;
  margin-top: 28px;
  padding: 4px 0;
  color: #fff;
  background: linear-gradient(to top, #0ba360 0%, #3cba92 100%);
  border-radius: 16px;
  float: left;
  text-align: center;
}

.main .update .button:hover {
  background: #8bc34a;
  cursor: pointer;
}

.main .list {
  margin-top: 15px;
}

.main .mode select {
  width: 660px;
  margin-top: 16px;
  margin-bottom: 4px;
  padding: 6px 12px;
  background-color: #fff;
  border: solid 1px #fff;
  border-radius: 6px;
}

.main .mode select:hover {
  border: solid 1px #bdbdbd;
}

.main .condition {
  margin: 10px 0;
  overflow: hidden;
}

.main .condition .button {
  width: 100px;
  height: 32px;
  margin-top: 28px;
  margin-right: 6px;
  padding: 4px 0;
  color: #fff;
  background: linear-gradient(to top, #0ba360 0%, #3cba92 100%);
  border-radius: 16px;
  float: left;
  text-align: center;
}

.main .onetouch {
  margin: 10px 0;
  overflow: hidden;
}

.main .onetouch .description,
.main .onetouch .button {
  float: left;
}

.main .onetouch .description {
  margin-right: 10px;
  padding: 4px 0;
}

.main .onetouch .button {
  height: 32px;
  margin: 0 4px;
  padding: 4px 10px;
  color: #424242;
  background-color: #e0e0e0;
  border-radius: 3px;
  text-align: center;
  cursor: pointer;
  transition: background-color 0.1s ease;
}

.main .onetouch .button:hover {
  background-color: #bdbdbd;
}

@media screen and (max-width: 850px) {
  .main .left,
  .main .right,
  .main .left .text,
  .main .right .text,
  .main .update .button,
  .main .mode select {
    width: 100%;
  }

  .main .right {
    margin-top: 8px;
  }

  .main .update .button {
    margin-top: 16px;
  }

  .main .onetouch .button {
    margin-bottom: 6px;
  }
}
</style>
