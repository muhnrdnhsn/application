<template>
  <article class="blog-card" :class="{'disable': isCurrentlyLabeled, 'blog-card': !isDelete, 'blog-delete': isDelete}">
    <img class="post-image" :class="{'post-image-disabled': isCurrentlyLabeled}" :src="imageURL">
    <div :class="{'article-details': !isCurrentlyLabeled, 'article-labeled': isCurrentlyLabeled}">
      <h3 class="post-title" :class="{'post-title-disabled': isCurrentlyLabeled, 'data-image-labeled': isCurrentlyLabeled}"> 
        {{ dataImageName }}
      </h3>
      <h4 v-if="isCurrentlyLabeled" class="labeled-title">
        This Image Is Currently Being Labeled
      </h4>
    </div>
  </article>
</template>

<script>
export default {
  props: {
    imageID: {
      type: Number,
      default: -1
    },
    imageURL: {
      type: String,
      default: ''
    },
    imageName: {
      type: String,
      default: ''
    },
    isCurrentlyLabeled: {
      type: Boolean,
      default: false
    },
    isDelete: {
      type: Boolean,
      default: false
    }
  },
  data () {
    return {
      dataImageName: ''
    }
  },
  mounted () {
    this.limitChars()
  },
  methods: {
    limitChars() {
      if (this.imageName.length > 20) {
        this.dataImageName = this.imageName.slice(0, 15) + '...'
      } else {
        this.dataImageName = this.imageName
      }
    }
  }
}  
</script>

<style scoped>
.blog-card, .blog-delete {
  background: #fff;

  -webkit-box-shadow: 2px 5px 5px 0px rgba(0,0,0,0.15);
  -moz-box-shadow: 2px 5px 5px 0px rgba(0,0,0,0.15);
  box-shadow: 2px 5px 5px 0px rgba(0,0,0,0.15);

  border-radius: 0.5rem;
  overflow: hidden;
  display: grid;
  grid-template-columns: 1fr 2fr;
  grid-template-rows: 1fr;

  cursor: pointer;
}
.blog-card:hover {
  background: #F0F0F0;
}

.blog-delete:hover {
  background: rgb(255, 0, 0, 0.3);
}

.disable {
  background-color: #F0F0F0;
  -webkit-box-shadow: 0 0 0 0px rgba(0,0,0,0.0);
  -moz-box-shadow: 0 0 0 0px rgba(0,0,0,0.0);
  box-shadow: 0 0 0 0px rgba(0,0,0,0.0);
}



.post-image {
  display: block;
  width: 100%;
  object-fit: cover;
  height: 100%;
}

.post-image-disabled {
  opacity: 0.5;
}

.article-details {
  padding: 1.7rem;
}

.article-labeled {
  margin-top: 0.5rem;
  padding-top: 0.275rem;
  padding-left: 0.5rem;
}

.data-image-labeled {
  margin-top: 0.5rem;
}

.labeled-title {
  font-size: 0.73rem;
  color: #1E889B;
  margin-top: 0.4rem;
}

.post-title {
  font-size: 0.875rem;
  line-height: 1;
  color: #1E889B;
  font-weight: bold;
  margin: 0 0 0 0;
}

.post-title-disabled {
  color: #D6D6D6;
}

</style>