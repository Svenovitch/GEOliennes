<template>
  <p class="title is-3 notification is-dark is-light"> {{ hellomessage }} </p>
  <div class="tile is-ancestor">
    <div class="tile is-6 is-vertical is-parent">
      <div class="content">
        <p class="title is-3">Notre équipe</p>
      </div>
      <div class="tile is-12 notification is-info is-light">
        <div v-for="collaborateur in team" :key="collaborateur" class="tile is-4">
          <article class="tile is-child">
            <p class="title is-4">{{collaborateur.nom}}</p>
            <p class="subtitle is-6">Ingénieur HES</p>
            <figure class="image is-1by1">
              <img :src = "collaborateur.avatar">
            </figure>
          </article>
        </div>
      </div>
    </div>
    <div class="tile is-6 is-vertical is-parent">
      <div class="content">
        <p class="title is-3">Le Projet</p>
      </div>
      <div class="tile is-child notification is-warning is-light">
        <article class="content has-text-justified has-text-black">
          <p>Les projets d’éoliennes font l’objet de nombreux débats. Ils sont souvent mal perçus par la population qui les rejettent presque systématiquement.</p>
          <p>Il est parfois difficile de se représenter l’impact qu’ont de tels projets sur notre territoire.
             Le but de ce géoportail est de fournir un support sur lequel représenter les éoliennes dans leur environnement, aussi bien sur une carte 2D qu’en 3 dimensions sur un globe virtuel.</p>
          <p>L’affichage sur la carte 2D permet de visualiser les éoliennes sur divers fonds de plan (carte nationale et imagerie aérienne par exemple).
             Pour chaque éolienne, un viewshed représentant les zones depuis lesquelles elles sont visibles peut être affiché.</p>
          <p>L’affichage 3D permet lui de visualiser un globe virtuel sur lequel les éoliennes, les bâtiments et les arbres sont représentés.
             Une fonction offre la possibilité à l’utilisateur de se projeter au sol, ce qui lui permet de s’immerger dans le paysage.</p>
        </article> 
      </div>
    </div>
  </div>
</template>

<script>
import Source from 'ol/source/Source'
export default {
  name: 'home',
  data(){
    return{
      hellomessage:"Bienvenue sur le portail de GEOliennes",
      team: [{nom:"Mathieu Allaz", avatar: require('../../assets/Mathieu.png')},
             {nom:"Rémi Girard", avatar: require('../../assets/Remi.png')},
             {nom:"Sven Fragnière", avatar: require('../../assets/Sven.png')}],
    }
  }
}
</script>