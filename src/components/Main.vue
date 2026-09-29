<template>
  <div class="main">
    <div class="information">
      <div class="left">
        <img src="../assets/img/info.svg" />
      </div>
      <div class="right">
        1～2つのMisskeyアカウントのフォローやフォロワーを比較するツールです。<br />
        テキストボックスへMisskeyのユーザとサーバ名 (@arkw@misskey.ioのようなフォーマット)
        を入力後、「更新」ボタンをクリックしてください。<br />
        表示条件を「ORで絞り込む」に切り替えるといずれかの条件に一致するアカウント、「ANDで絞り込む」に切り替えると全てのチェックボックスに一致するアカウントのみ表示します。
      </div>
    </div>
    <div class="input">
      <div class="left">
        <div class="description">アカウントA (例: @arkw@misskey.io)</div>
        <input type="text" class="text" v-model="inputModel[0]" />
      </div>
      <div class="right">
        <div class="description">アカウントB (例: @arkw@mi.arkw.work)</div>
        <input type="text" class="text" v-model="inputModel[1]" />
      </div>
      <div class="update">
        <div class="button" @click="update">更新</div>
      </div>
    </div>
    <div class="mode">
      <div class="description">表示条件</div>
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
      <div v-for="user in listShow" v-bind:key="user.id">
        <User
          :avatar="user.avatar"
          :name="user.name"
          :id="user.id"
          :url="user.url"
          :value="user.value"
        />
      </div>
      <div class="button" v-if="isMoreButton" @click="addItem">
        <span>もっと見る</span>
        <img src="../assets/img/more.svg" alt="" />
      </div>
    </div>
    <div class="cover" v-if="isLoading">
      <div class="dialog"><div class="loader"></div></div>
    </div>
  </div>
</template>

<script setup lang="js">
import { reactive, ref } from 'vue'
import axios from 'axios'
import Checkbox from './Checkbox.vue'
import User from './User.vue'
const inputModel = ref([''], [''])
const list = new Array()
const listShow = ref(new Array())
const mode = ref('')
const pointer = ref(0)
const isMoreButton = ref(false)
const isLoading = ref(false)
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

const sleep = (ms) => {
  new Promise((resolve) => setTimeout(resolve, ms))
}

const getList = async (host, url, id, max, type) => {
  const output = new Array()
  const set = new Set()
  let count = 0
  let next = null
  for (let i = 0; i < max / 100 + 2; i++) {
    let response
    if (next != null) {
      response = await axios.post('https://' + host + url, {
        userId: id,
        limit: 100,
        untilId: next,
      })
    } else {
      response = await axios.post('https://' + host + url, {
        userId: id,
        limit: 100,
      })
    }
    for (let j = 0; j < response.data.length; j++) {
      if (j == response.data.length - 1) {
        next = response.data[j].id
      } else {
        let id
        if (type == 'followee') {
          if (response.data[j].followee.host != null) {
            id = '@' + response.data[j].followee.username + '@' + response.data[j].followee.host
          } else {
            id = '@' + response.data[j].followee.username + '@' + host
          }
          if (set.has(id) == false) {
            output.push({
              id: id,
              avatar: response.data[j].followee.avatarUrl,
              name: response.data[j].followee.name,
              url: response.data[j].followee.url,
            })
            set.add(id)
          }
        } else if (type == 'follower') {
          if (response.data[j].follower.host != null) {
            id = '@' + response.data[j].follower.username + '@' + response.data[j].follower.host
          } else {
            id = '@' + response.data[j].follower.username + '@' + host
          }
          if (set.has(id) == false) {
            output.push({
              id: id,
              avatar: response.data[j].follower.avatarUrl,
              name: response.data[j].follower.name,
              url: response.data[j].follower.url,
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
    userhost[2],
    '/api/users/followers',
    user.data[0].id,
    stats.data.followingCount,
    'follower',
  )
  output.following = await getList(
    userhost[2],
    '/api/users/following',
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

const addList = (id, avatar, name, url, index, type) => {
  let position = list.findIndex((el) => el.id === id)
  if (position == -1) {
    list.push({
      id: id,
      avatar: avatar,
      name: name,
      url: url,
      value: [
        {
          followers: false,
          following: false,
          url: null,
        },
        {
          followers: false,
          following: false,
          url: null,
        },
      ],
    })
    position = list.length - 1
  }
  if (type == 'followee') {
    list[position].value[index].following = true
    list[position].value[index].url =
      'https://' + inputModel.value[index].split('@').pop() + '/' + id
  } else if (type == 'follower') {
    list[position].value[index].followers = true
    list[position].value[index].url =
      'https://' + inputModel.value[index].split('@').pop() + '/' + id
  }
  return
}

const update = async () => {
  isLoading.value = true
  list.splice(0)
  for (let i = 0; i < 2; i++) {
    if (isValid(inputModel.value[i]) == true) {
      const data = await getUser(inputModel.value[i])
      for (let j = 0; j < data.following.length; j++) {
        addList(
          data.following[j].id,
          data.following[j].avatar,
          data.following[j].name,
          data.following[j].url,
          i,
          'followee',
        )
      }
      await sleep(100)
      for (let j = 0; j < data.followers.length; j++) {
        addList(
          data.followers[j].id,
          data.followers[j].avatar,
          data.followers[j].name,
          data.followers[j].url,
          i,
          'follower',
        )
      }
      await sleep(100)
    }
  }
  changeList()
  isLoading.value = false
}

const changeList = () => {
  listShow.value = []
  if (mode.value === '') {
    isShow[0].followers = true
    isShow[0].following = true
    isShow[1].followers = true
    isShow[1].following = true
  }
  pointer.value = 0
  addItem()
}

const addItem = async () => {
  let hit = 0
  for (let i = 0; i < list.length; i++) {
    const item = list[pointer.value]
    let add = false
    if (mode.value === 'or') {
      add =
        (isShow[0].followers && item.value[0].followers) ||
        (isShow[0].following && item.value[0].following) ||
        (isShow[1].followers && item.value[1].followers) ||
        (isShow[1].following && item.value[1].following)
    } else if (mode.value === 'and') {
      add =
        isShow[0].followers === item.value[0].followers &&
        isShow[0].following === item.value[0].following &&
        isShow[1].followers === item.value[1].followers &&
        isShow[1].following === item.value[1].following
    } else {
      add = true
    }
    if (add) {
      listShow.value.push({ ...item })
      hit++
    }
    pointer.value++
    if (pointer.value >= list.length) {
      break
    }
    if (hit >= 50) {
      break
    }
    await sleep(10)
  }
  if (pointer.value + 1 < list.length) {
    isMoreButton.value = true
  } else {
    isMoreButton.value = false
  }
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

<style src="../assets/sass/components/Main.scss" lang="scss" scoped></style>
