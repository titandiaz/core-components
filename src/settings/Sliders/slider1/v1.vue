<template>
  <div class="form_list">
    <section class="settingBanner" v-for="(banner, index) in bannersData">
      <img
        :src="getBanner(banner.ruta_banner)"
        :alt="banner.titulo"
        class="banner_photo">
      <div class="input-area">
        <el-input placeholder="Titulo" v-model="banner.titulo">
          <template slot="prepend">
            <icon-base icon-name="text"><icon-text /></icon-base>
          </template>
        </el-input>
      </div>
      <div class="input-area">
        <el-input placeholder="Descripción" v-model="banner.descripcion">
          <template slot="prepend"><icon-base icon-name="align-justify"><align-justify /></icon-base></template>
        </el-input>
      </div>
      <div class="input-area">
        <el-input placeholder="Url de redirección" v-model="banner.redireccion">
          <template slot="prepend">
            <icon-base icon-name="links"><icon-links /></icon-base>
          </template>
        </el-input>
      </div>
      <div class="input-area">
        <el-select placeholder="Posición del contenido" v-model="banner.posicion">
          <el-option
            v-for="position in positions"
            :label="position.label"
            :value="position.value"
            :key="position.value">
          </el-option>
        </el-select>
      </div>
      <div class="settingBanner_actions">
        <el-button @click="updateBanner(banner)">Guardar</el-button>
        <el-button type="danger" icon="el-icon-delete" @click="deleteBanner(banner, index)">Eliminar</el-button>
      </div>
    </section>
    <section class="settingBanner" v-if="bannersData.length < 3">
      <div class="upload-area">
        <label for="uploadBanner" class="upload">
          <icon-base icon-name="cloud-up" icon-color="#FFF"><cloud-up /></icon-base>Subir imagen
        </label>
        <button class="banner_unplash" @click="openUnplash">
          Explora imágenes gratis
        </button>
        <input type="file" id="uploadBanner" v-on:change="uploadBanner">
      </div>
      <div class="input-area">
        <el-input placeholder="Titulo" v-model="titulo">
          <template slot="prepend">
            <icon-base icon-name="text"><icon-text /></icon-base>
          </template>
        </el-input>
      </div>
      <div class="input-area">
        <el-input placeholder="Descripción" v-model="descripcion">
          <template slot="prepend">
            <icon-base icon-name="align-justify"><align-justify /></icon-base>
          </template>
        </el-input>
      </div>
      <div class="input-area">
        <el-input placeholder="Url de redirección" v-model="redireccion">
          <template slot="prepend">
            <icon-base icon-name="links"><icon-links /></icon-base>
          </template>
        </el-input>
      </div>
      <div class="settingBanner_actions">
        <el-button>Guardar</el-button>
      </div>
    </section>
  </div>
</template>

<script>
import axios from 'axios';
import AngleLeft from '../../Icons/AngleLeft.vue'
import IconText from '../../Icons/Text.vue'
import AlignJustify from '../../Icons/AlignJustify.vue'
import IconLinks from '../../Icons/Links.vue'
import CloudUp from '../../Icons/CloudUp.vue'

export default {
  name: 'koSliderSetting1',
  components: { AngleLeft, IconText, AlignJustify, IconLinks, CloudUp },
  data() {
    return {
      titulo: '',
      descripcion: '',
      redireccion: '',
      newBanner: false,
      positions: [
        {
          label: 'Izquierda',
          value: 'left'
        },
        {
          label: 'Centrado',
          value: 'center'
        },
        {
          label: 'Derecha',
          value: 'right'
        }
      ]
    }
  },
  computed: {
    settingData () {
      return this.$store.state.settingData
    },
    bannersData() {
      return this.$store.state.banners;
    }
  },
  methods: {
    openUnplash() {
      this.$store.state.unplash.status = true;
    },
    uploadBanner(event) {
      this.$cropper.upload({
        type: 'Banner',
        ratio: 12/4,
        file: event.target.files[0],
        desc: 'Peso maximo del banner 20M y tamaño de 1200px X 400px'
      }).then((response) => {
        this.createBanner(response)
      })
    },
    createBanner(blob) {
      let config = {
        headers:
          {
            'content-type': 'multipart/form-data',
            'Authorization': `Bearer ${this.$configKomercia.accessToken}`,
            'Access-Control-Allow-Origin': '*'
          }
      }
      let params = new FormData()
      params.append('banner', blob)
      params.append('order', this.bannersData.length + 1)
      axios.post(`${this.$configKomercia.url}/api/admin/tienda/banners/cargar`, params, config).then((response) => {
        this.$store.state.banners.push(response.data.data)
        this.$cropper.complete()
      }).catch((error) => {
        if (error) {
          this.$cropper.complete()
        }
      })
    },
    deleteBanner(banner, index) {
      let config = {
        headers:
          {
            'content-type': 'application/json',
            'Authorization': `Bearer ${this.$configKomercia.accessToken}`,
            'Access-Control-Allow-Origin': '*'
          }
      }
      let params = {
        id: banner.id
      }
      axios.post(`${this.$configKomercia.url}/api/admin/tienda/banners/eliminar`, params, config).then((response) => {
        this.bannersData.splice(index, 1)
      })
    },
    updateBanner (banner) {
      let config = {
        headers:
          {
            'content-type': 'application/json',
            'Authorization': `Bearer ${this.$configKomercia.accessToken}`,
            'Access-Control-Allow-Origin': '*'
          }
      }
      let params = {
        titulo: banner.titulo,
        descripcion: banner.descripcion,
        redireccion: banner.redireccion,
        posicion: banner.posicion
      }
      axios.put(`${this.$configKomercia.url}/api/admin/tienda/banners/${banner.id}`, params, config).then((response) => {
      })
    },
    closeSetting() {
      this.$store.state.settingData = null;
    },
    getBanner(banner) {
      return `https://api2.komercia.co/banners/${banner}`
    }
  }
}
</script>

<style scoped>
.sectionBar {
  position: absolute;
  top: 0;
  width: 100%;
  background-color: #e9ecef;
  z-index: 3;
}
.sectionBar_header {
  width: 100%;
  height: 50px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 20px;
  background-color: #fff;
  box-sizing: border-box;
  box-shadow: 0 0 0 1px rgba(63, 63, 68, 0.05),
    0 1px 3px 0 rgba(63, 63, 68, 0.15);
}
.sectionBar_header p {
  /*color: #FFF;*/
}
.sectionBar_header span {
  flex: 1;
  display: flex;
  align-items: center;
}
.sectionBar_header span.angle-left {
  cursor: pointer;
}
.form_list {
  box-sizing: border-box;
  padding: 10px 5px;
  overflow: auto;
}
.settingBanner {
  padding: 20px 0;
  border-bottom: 1px solid rgba(0, 0, 0, 0.1);
}
.banner_photo {
  width: 100%;
  border: 1px solid #c4cdd5;
}
.input-area {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  margin: 5px 0;
}
.input-area p {
  margin-bottom: 5px;
  font-size: 14px;
  color: #333;
}
.upload-area {
  width: 100%;
  height: 120px;
  display: grid;
  justify-content: center;
  align-content: center;
  grid-row-gap: 10px;
  border: 1px solid #c4cdd5;
  background-color: #f4f6f8;
}

#uploadBanner {
  visibility: hidden;
  opacity: 0;
  width: 0;
  height: 0;
  display: none;
}
.upload {
  background-color: #0f9380;
  color: #fff;
  padding: 10px;
  cursor: pointer;
}
.upload img {
  width: 100%;
}
.upload svg {
  fill: #fff;
}
.banner_unplash {
  border-style: none;
  background-color: transparent;
  text-decoration: underline;
}
.newBanner_action {
  display: flex;
  align-items: center;
  justify-content: space-around;
  background-color: #f5f7fa;
  box-sizing: border-box;
  border: 1px solid #dcdfe6;
  border-radius: 4px;
  padding: 5px;
  color: #909399;
  margin-top: 10px;
  font-size: 14px;
}
</style>
