<template>
   <div class="card">
      <header class="card-header">
         <p class="card-header-title">List of photos</p>
         <a @click="showCard = !showCard" aria-label="more options" class="card-header-icon">
            <b-icon :icon="showCard ? 'chevron-up' : 'chevron-down'"></b-icon>
         </a>
      </header>
      <div class="card-content" v-if="showCard">
         <b-field label="Series" v-if="isMobile">
            <b-select expanded placeholder="Available series" v-model="selectedSeriesMobile">
               <option :key="aSeries.series.id" :value="aSeries.series" v-for="aSeries in series">
                  {{ aSeries.series.city }}
               </option>
            </b-select>
         </b-field>
         <div class="columns is-mobile is-multiline">
            <div class="column is-one-third is-one-quarter-mobile is-hidden-mobile">
               <b-field label="Series" v-if="!isMobile">
                  <b-select
                     expanded
                     multiple
                     native-size="10"
                     placeholder="Available series"
                     v-model="selectedSeries"
                  >
                     <option :key="aSeries.series.id" :value="aSeries.series" v-for="aSeries in series">
                        {{ aSeries.series.city }}
                     </option>
                  </b-select>
               </b-field>
            </div>
            <div :class="{ 'is-two-thirds': !isMobile, 'is-full': isMobile }" class="column">
               <b-field label="Photos"></b-field>
               <div id="gallery">
                  <div class="gallery">
                     <img
                        @click="sendInfoModal(photo.photo.id)"
                        v-for="photo in photosBySeries"
                        :src="photo.photo.url"
                        alt="Photo"
                        width="100%"
                        height="auto"
                        class="gallery-img"
                     />
                  </div>
               </div>
               <!--          <div class="columns is-mobile is-multiline">-->
               <!--            <div-->
               <!--              :class="{ 'is-one-third': !isMobile, 'is-half': isMobile }"-->
               <!--              :key="photo.photo.id"-->
               <!--              class="column"-->
               <!--              v-for="photo in photosBySeries"-->
               <!--            >-->
               <!--              <figure class="image">-->
               <!--                <img-->
               <!--                  :src="photo.photo.url"-->
               <!--                  @click="sendInfoModal(photo.photo.id)"-->
               <!--                  alt="Photo"-->
               <!--                />-->
               <!--              </figure>-->
               <!--            </div>-->
               <!--          </div>-->
            </div>
         </div>
         <b-modal :active.sync="showModal" :can-cancel="['x', 'outside']" has-modal-card>
            <photos-info :selected-photo="selectedPhoto"></photos-info>
         </b-modal>
      </div>
   </div>
</template>

<script>
import BTable from 'buefy/src/components/table/Table';
import BTableColumn from 'buefy/src/components/table/TableColumn';
import BIcon from 'buefy/src/components/icon/Icon';
import BSwitch from 'buefy/src/components/switch/Switch';
import BField from 'buefy/src/components/field/Field';
import BSelect from 'buefy/src/components/select/Select';
import BButton from 'buefy/src/components/button/Button';
import BModal from 'buefy/src/components/modal/Modal';
import BTooltip from 'buefy/src/components/tooltip/Tooltip';
import PhotosInfo from './photosInfo';

export default {
   name: 'photosTable',
   components: {
      PhotosInfo,
      BTooltip,
      BModal,
      BButton,
      BSelect,
      BField,
      BSwitch,
      BIcon,
      BTableColumn,
      BTable,
   },
   data() {
      return {
         perPage: 5,
         isPaginated: true,
         hasMobileCards: true,
         showCard: true,
         showModal: false,
         selectedSeries: [],
         selectedSeriesMobile: null,
         selectedPhoto: {},
      };
   },
   computed: {
      photosBySeries() {
         if (this.isMobile && this.selectedSeriesMobile !== null)
            return this.$store.getters.PHOTOS_BY_SERIES(this.selectedSeriesMobile.id);
         else if (this.selectedSeries.length === 0) {
            return [];
         } else {
            return this.$store.getters.PHOTOS_BY_SERIES(this.selectedSeries[0].id);
         }
      },
   },
   methods: {
      async sendInfoModal(photoId) {
         await this.$store.dispatch('GET_PHOTO', photoId).then((response) => {
            if (response.success) this.selectedPhoto = response.data.photo;
            else console.error(response.message);
         });
         this.showModal = true;
      },
      handleResize() {
         this.window.width = window.innerWidth;
         this.window.height = window.innerHeight;
         this.isMobile = this.window.width < 1024;
      },
   },
   destroyed() {
      window.removeEventListener('resize', this.handleResize);
   },
};
</script>

<style scoped>
img {
   cursor: pointer;
}

#gallery {
   min-height: 360px;
   max-height: 360px;
   overflow-y: scroll;
}

#gallery::-webkit-scrollbar-track {
   -webkit-box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.3);
   background-color: #f5f5f5;
}

#gallery::-webkit-scrollbar {
   width: 5px;
   background-color: #f5f5f5;
}

#gallery::-webkit-scrollbar-thumb {
   background-color: #990019;
   background-image: -webkit-linear-gradient(
      45deg,
      rgba(255, 255, 255, 0.2) 25%,
      transparent 25%,
      transparent 50%,
      rgba(255, 255, 255, 0.2) 50%,
      rgba(255, 255, 255, 0.2) 75%,
      transparent 75%,
      transparent
   );
}

.gallery {
   -webkit-column-count: 3;
   -moz-column-count: 3;
   column-count: 3;
   -webkit-column-gap: 10px;
   -moz-column-gap: 10px;
   column-gap: 10px;
   margin-top: 10px;
   overflow: hidden;
}

.gallery img {
   width: 100%;
   height: auto;
   transition: 500ms;
   margin-bottom: 10px;
   opacity: 1;
   page-break-inside: avoid; /* For Firefox. */
   -webkit-column-break-inside: avoid; /* For Chrome & friends. */
   break-inside: avoid; /* For standard browsers like IE. :-) */
}

.gallery img:hover {
   opacity: 0.8;
}

@media screen and (max-width: 767px) {
   .gallery {
      -webkit-column-count: 2;
      -moz-column-count: 2;
      column-count: 2;
   }
   .gallery div {
      margin: 0;
      width: 200px;
   }
}

@media screen and (max-width: 479px) {
   .gallery {
      -webkit-column-count: 1;
      -moz-column-count: 1;
      column-count: 1;
   }
   .gallery div {
      margin: 0;
      width: 200px;
   }
}
</style>
