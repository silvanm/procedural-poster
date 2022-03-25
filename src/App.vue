<template>
  <div class="polaroids">
    <div class="polaroid" v-for="p in people" :key="p" :style="{ transform: `rotate(${Math.random()*10-5}deg)` } ">
      <img v-if="p.avatar" :src="p.avatar">
      <img v-else :src="'https://robohash.org/' + p.name">
      <div class="signature" :style="{ backgroundImage: `url(/signatures/${p.name}.jpg)` } "></div>
    </div>

    <div class="greeting">
      <p class="logo">Mühlemann&Popp</p>
      <p class="middle">wishes you a</p>
      <p class="happy_wedding">Happy Wedding!</p>
    </div>
  </div>

</template>

<script>
import {people as peopleStr} from './users'
import axios from 'axios'

const picOverrides = [ 'viktor.sobor' ]

export default {
  name: 'App',
  data() {
    return {
      people: []
    }
  },
  computed: {
    rotation() {
      console.log('rotate')
      return `rotate(${Math.random() * 10 - 5})`
    }
  },
  async mounted() {

    // poor man's pagination
    const avatars = {}

    for (let page=1; page<=2; page++) {
      const config = {
        method: 'get',
        url: `https://app.peopleforce.io/api/public/v1/employees?status=active&page=${page}`,
        headers: {
          'X-API-KEY': process.env.VUE_APP_PEOPLEFORCE_API_KEY
        }
      };

      const peopleforceData = await axios(config)

      peopleforceData.data.data.forEach((o) => {
        if (o.email) {
          let userId = o.email.replace(/@muehlemann-popp.ch/, '')
          if (picOverrides.indexOf(userId) >= 0) {
            avatars[userId] = '/' + userId + '.jpg'
          } else {
            avatars[userId] = o.avatar_url
          }
        }
      })
    }

    const peopleNames = peopleStr.split("\n").sort()
    this.people = peopleNames.map((s) => {
      return {
        isText: false,
        name: s,
        avatar: avatars[s]
      }
    })
  }
}
</script>

<style lang="scss">

$cols: 7;
$width: 40cm;
$height: 52cm;
$padding: 0.4cm;
$margin: 0.5cm;

@import url('https://fonts.googleapis.com/css2?family=Great+Vibes&family=Open+Sans:wght@300;700&display=swap');

body {

}

.polaroids {
  background-color: #6c4c76;
  width: $width;
  height: $height;
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  align-content: flex-start;
  padding: 2cm 1cm 1cm 2cm;
}

.polaroid {
  background-color: white;
  padding: $padding $padding $padding/3 $padding;
  margin: $margin;
  -webkit-box-shadow: 5px 5px 15px 5px rgba(0, 0, 0, 0.56);
  box-shadow: 5px 5px 15px 5px rgba(0, 0, 0, 0.56);

  img, .polaroid div {
    width: ((($width)-2*$margin)/$cols)-(2*$padding)-2.2*$margin;
  }

  .signature {
    background-size: contain;
    background-position: center;
    background-repeat: no-repeat;
    margin-top: 0.2cm;
    height: 1.5cm;
  }
}

.greeting {
   //flex-grow: 2;
  margin: $margin;
  width: 830px;
  /*width: $width/$cols*1-$margin*4;*/
  padding-top: 10mm;
  color: white;
  text-align: center;
  font-size: 1cm;

  p {
    font-family: 'Great Vibes', cursive;
    margin: 0;
    line-height: 140%;

    &.happy_wedding {
      font-size: 180%;
    }
  }

  p.logo {
    font-family: 'Open Sans', sans-serif;
    font-weight: 700;
    letter-spacing: -0.5mm;
    margin-bottom: 10px;
  }
}

</style>
