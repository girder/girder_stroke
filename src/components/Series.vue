<template lang="pug">
v-card.cinema-wrapper(light)
  .cinema-img
  .subheading {{ series.name }}
  v-menu(offset-y)
    v-btn(slot="activator")
     | Actions
     v-icon mdi-chevron-down
    v-list.py-0(dense)
      v-list-tile(:to="`glance/${series._id}`")
        v-list-tile-title Open with ParaView Glance
</template>

<script>
import QueryDataModel from 'paraviewweb/src/IO/Core/QueryDataModel';
import MouseHandler from 'paraviewweb/src/Interaction/Core/MouseHandler';
import 'setimmediate'; // sigh...

export default {
  inject: ['girderRest'],
  props: {
    series: {
      type: Object,
      required: true,
    },
  },
  mounted() {
    const base = `/item/${this.series._id}/dicom_thumbnail/`;

    this.girderRest.get(`${base}index.json`).then(({ data }) => {
      const container = this.$el.querySelector('.cinema-img');
      this.qdm = new QueryDataModel(data, GIRDER_API_ROOT + base);
      this.qdm.onDataChange(({ image }) => {
        container.innerHTML = '';
        container.appendChild(image.image);
      });
      this.mouseHandler = new MouseHandler(container, { preventDefault: false });
      this.qdm.fetchData();
      this.mouseHandler.attach(this.qdm.getMouseListener());
    }).catch(() => {
      // TODO show the "no thumbnail" message
    });
  },
  destroyed() {
    if (this.mouseHandler) {
      this.mouseHandler.destroy();
    }
    if (this.qdm) {
      this.qdm.destroy();
    }
  },
};
</script>

<style lang="scss" scoped>
.cinema-wrapper {
    display: inline-block;
    text-align: center;
    background-color: #f1f1f1 !important;
    max-width: 272px;
    padding: 8px;

    .subheading {
        text-overflow: ellipsis;
        white-space: nowrap;
        overflow: hidden;
    }
}

.cinema-img {
    width: 256px;
    height: 256px;
    background-color: black;
}

</style>

<style lang="scss">
.cinema-img img {
     -webkit-user-drag: none;
}
</style>
