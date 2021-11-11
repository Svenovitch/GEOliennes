<template>
  <p class="title"> {{ hellomessage }} </p>
</template>

<script>
export default {
  name: 'home',
  data(){
    return{
      hellomessage:"Bienvenue sur le portail de GEOliennes"
    }
  }
}
</script>