<template>
  <client-only>
    <Label
      :key="dataset"
      title="Edit Labeled Images"
      viewer-u-r-l="/viewer/index-edit"
      :is-labeled="isLabeled"
      :dataset="dataset"
    >
      <Dropdown :dropdown-value="dataset" @onDatasetChanged="changeDataset" />
    </Label>
  </client-only>
</template>

<script>
import Label from '~/components/label/Label'
import Dropdown from '~/components/dropdown/Dropdown'
export default {
  components: {
    Label,
    Dropdown
  },
  data () {
    return {
      isLabeled: true,
      dataset: ''
    }
  },
  mounted () {
    if (this.$route.query.dataset) {
      this.dataset = this.$route.query.dataset
    }
  },
  methods: {
    changeDataset (newDataset) {
      this.dataset = newDataset
    }
  }
}
</script>