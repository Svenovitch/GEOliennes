<template>
  <p class="title is-2"> {{ hellomessage }} </p>
  <div class="tile is-ancestor">
    <div class="tile is-6 is-vertical">
      <div class="tile is-parent">
        <article class="tile is-child">
          <p class="title">Notre équipe</p>
        </article>
      </div>
      <div class="tile is-12 notification is-info is-light">
        <div v-for="collaborateur in team" :key="collaborateur" class="tile is-4">
          <article class="tile is-child">
            <p class="title is-4">{{collaborateur}}</p>
            <p class="subtitle is-6">Ingénieur HES</p>
            <figure class="image is-3by4">
              <!-- <img src = "../../assets/Sven.jpg"> -->
            </figure>
          </article>
        </div>
      </div>
    </div>
    <div class="tile is-6 is-vertical">
      <div class="tile is-parent">
        <article class="tile is-child">
          <div class="content">
            <p class="title">Le Projet</p>
            <div class="content">
              <p class="is-small">dfsdgsfdfsfdhdfhfsdsfsfwgdwgdsgdsgdsgdsgdsgdsgdsgreefehehrfdgazre r zrezeqe  utuaet utau tr ur srewhbfhdsfv hvhjfvhsavf vfjsavfhufvh</p>
            </div>
          </div>
        </article>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'home',
  data(){
    return{
      hellomessage:"Bienvenue sur le portail de GEOliennes",
      team: ["Mathieu Allaz", "Rémi Girard", "Sven Fragnière"]
    }
  }
}
</script>