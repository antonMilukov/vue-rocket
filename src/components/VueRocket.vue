<template>
  <div>
    <h1>{{ intro }}</h1>

    <p>
        <input id="username" type="text" v-model="username"> Username
    </p>

    <p>
        <input id="password" type="password" v-model="password"> Password
    </p>

    <div v-if="roomId">
      <h3>{{ roomId }}</h3>
      <ul>
        <li v-for="(message, $index) in messages" :key="$index">{{ message }}</li>
      </ul>
    </div>

  </div>
</template>

<script>
export default {
  name: 'VueRocket',
  data: ()=>{
    return {
      intro: "RocketVue component by Anton Milukov",
      username: '',
      password: '',
      messages: ['123', '456', '789'],
      roomId: '1',
    };
  },
}
</script>