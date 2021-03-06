<template>
  <client-only>
    <div ref="imageWrapper" class="viewer-background">
      <div class="flex-viewer">
        <Toolbar
          @onIconClick="setBoxAction($event)"
        />
        <div class="viewer-wrapper center center-horizontal" @mousedown="stopDrawingBox">
          <div id="image">
            <img
              v-if="dataReady"
              ref="image"
              draggable="false"
              class="image"
              :style="{ cursor: cssCursor}"
              :src="image.url"
              @mousedown="onMouseDownHandler"
              @mousemove="changeBox"
              @mouseup="stopDrawingBox"
            >
            <Box
              v-if="drawingBox.active"
              :b-width="drawingBox.width"
              :b-height="drawingBox.height"
              :b-top="drawingBox.top"
              :b-left="drawingBox.left"
            />
            <div v-for="i in Object.keys(boxes)" :key="i">
              <Box
                v-if="boxes[i]"
                :b-width="boxes[i].width"
                :b-height="boxes[i].height"
                :b-top="boxes[i].top"
                :b-left="boxes[i].left"
                :b-active="i === activeBoxIndex"
                :b-index="parseInt(i)"
                :b-content="boxes[i].content"
                :b-action="actionName"
                :can-delete="canDelete"
                :suggest-type="'edit'"
                @onStopResize="changeBoxAttribute($event, i)"
                @onDelete="deleteBox(i)"
                @onSelect="makeCurrentBoxActive(i)"
                @onDisableForm="changeBoxContent($event, i)"
                @onEnableForm="makeCurrentBoxActive(i)"
              />
            </div>
          </div>
        </div>
        <div class="button-block">
          <div class="btn-close-section">
            <button
              type="button"
              class="btn-label-no-border btn-sm btn-light btn-close-text mt-2"
              aria-label="Close"
              @click="closeViewer()"
            > 
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
        </div>
      </div>
      <div class="btn-save-section">
        <button
          type="button"
          class="btn-label-no-border btn-lg btn-dark btn-text"
          @click="saveImage"
        >
          Save Image
        </button>
      </div>
    </div>
  </client-only>
</template>

<script>
import Box from '~/components/label/Box'
import Toolbar from '~/components/label/Toolbar'
import { Cursors } from '~/mixins/label/getCursorPosition'
export default {
  components: {
    Box,
    Toolbar
  },
  data () {
    return {
      drawingBox: {
        active: false,
        top: 0,
        left: 0,
        height: 0,
        width: 0,
        content: ''
      },
      actionName: 'add-box',
      activeBoxIndex: -1,
      cssCursor: 'cell',
      boxes: {},
      image: {
        id: -1,
        url: ''
      },
      dataset: '',
      canDelete: true,
      labelCount: 0,
      timer: '',
      previouslyCreatedBox: {},
      dataReady: true,
      isEdited: false,
      boxesCount: 0,
      deletedBoxesCount: 0,
      deletedBoxes: {},
      deletedBoxKey: {}
    }
  },
  async mounted () {

    window.addEventListener("beforeunload", async (event) => {
      await this.closeViewer()
      event.returnValue= "You have unsaved changes."
    })
    await this.startHeartBeat()
    this.timer = setInterval(this.startHeartBeat, 90000)

    this.image.url = this.$route.query.url
    this.image.id = this.$route.query.id
    this.dataset = this.$route.query.dataset
    await this.drawAllBox()
    this.dataReady = false
    this.dataReady = true
  },
  async beforeDestroy () {
    // window.removeEventListener("beforeunload", true)
    await this.closeViewer()
  },
  methods: {
    async startHeartBeat() {
      var url = '/api/accesscontrol/requestaccess/' + parseInt(this.$route.query.id)
      // alert(url)
      try {
        await this.$axios.get(url).catch((error) => console.error(error))
      } catch (error) {
        this.showFailedAlert("An error occured", error)
        await this.closeViewer()
      }
    },
    setBoxAction (iconName) {
      this.actionName = iconName
      this.activeBoxIndex = -1
      switch (iconName) {
      case 'add-box':
        this.cssCursor = 'cell'
        break
      case 'resize-box':
        this.cssCursor = 'default'
        break
      case 'delete-box':
        this.cssCursor = 'default'
        break
      }
    },
    async getAllLabels () {
      var url = '/api/label/imagequery/' + this.image.id
      var response = await this.$axios.get(url).catch( error => console.error(error))
      if(response && response.status === 200) {
        return response.data.data
      } else {
        return null
      }
    },
    async getContentName (id) {
      var url = '/api/content/' + id
      var response = await this.$axios.get(url).catch( error => console.error(error))
      if(response.status === 200) {
        return response.data.data.content_name
      } else {
        return null
      }
    },
    async drawAllBox () {
      var allLabels = await this.getAllLabels()
      console.log('labeleeees: ', allLabels)
      if (allLabels) {
        var labeledCount = Object.keys(allLabels).length
        var imageAttributes = {
          screenWidth: this.$refs.image.clientWidth,
          screenHeight: this.$refs.image.clientHeight,
          realWidth: this.$refs.image.naturalWidth,
          realHeight: this.$refs.image.naturalHeight
        }
        while(labeledCount > this.labelCount){
          this.previouslyCreatedBox[this.labelCount + 1] = allLabels[this.labelCount].label_id
          var contentName = await this.getContentName(allLabels[this.labelCount].label_content_id)
          
          var screenImagesAttr = this.getScreenAttributes(imageAttributes, allLabels[this.labelCount].label_width, allLabels[this.labelCount].label_height, allLabels[this.labelCount].label_x_center, allLabels[this.labelCount].label_y_center)
          let newBox = {
            width: screenImagesAttr.width,
            height: screenImagesAttr.height,
            left: screenImagesAttr.left,
            top: screenImagesAttr.top,
            content: contentName, 
            label_id: allLabels[this.labelCount].label_id
          }
          this.labelCount++
          this.boxes[this.labelCount] = newBox
          this.boxesCount++
          console.log('prev: ', this.boxes)
          // 
          this.changeBoxContent(newBox.content, this.labelCount)
          // this.makeCurrentBoxActive(this.labelCount)
          this.resetDrawingBox()
        }
      }
      
      
    },
    async closeViewer () {
      try {
        await this.deleteImageAccessControlByImageID(this.image.id)
      } catch (error) {
        console.error(error)
      }
      clearInterval(this.timer)
      this.$router.push({ path: '/main/edit', query: {dataset: this.dataset }})
    },
    async deleteImageAccessControlByImageID (imageID) {
      var url = '/api/accesscontrol/' + imageID
      try {
        var response = await this.$axios.delete(url)
        return response.data.status
      } catch (error) {
        console.error(error)
        throw error
      }
    },
    onMouseDownHandler (e) {
      if (this.drawingBox.active) {
        this.stopDrawingBox()
      } else {
        this.startDrawingBox(e)
      }
    },
    startDrawingBox (e) {
      
      this.drawingBox = {
        width: 0,
        height: 0,
        left: Cursors.getLeftCursor(e),
        top: Cursors.getTopCursor(e),
        active: true
      }
    },
    changeBox (e) {
      if (this.drawingBox.active) {
        this.drawingBox.width = Cursors.getLeftCursor(e) - this.drawingBox.left
        this.drawingBox.height = Cursors.getTopCursor(e) - this.drawingBox.top
      }
    },
    changeBoxAttribute (attribute, index) {
      var idxBox = index
      this.boxes[idxBox].left = attribute.bLeft
      this.boxes[idxBox].top = attribute.bTop
      this.boxes[idxBox].width = attribute.bWidth
      this.boxes[idxBox].height = attribute.bHeight
      
    },
    makeCurrentBoxActive (activeBoxIndex) {
      this.activeBoxIndex = activeBoxIndex
    },
    deleteBox (index) {
      
      
      this.deletedBoxesCount++
      this.deletedBoxKey[this.deletedBoxesCount] = this.boxes[index].label_id
      
      this.deletedBoxes[this.deletedBoxesCount] = this.boxes[index]
      delete this.boxes[index]
      this.activeBoxIndex = -1
    },
    changeBoxContent (content, index) {
      var idxBox = index
      this.boxes[idxBox].content = content
    },
    stopDrawingBox () {
      if (this.drawingBox.active) {
        if (this.drawingBox.width > 5) {
          let newBox = {
            width: this.drawingBox.width,
            height: this.drawingBox.height,
            left: this.drawingBox.left,
            top: this.drawingBox.top,
            content: this.drawingBox.content
          }
          this.labelCount++
          this.boxes[this.labelCount] = newBox
          this.makeCurrentBoxActive(this.labelCount)
          this.resetDrawingBox()
          this.boxesCount++
        }
      }
    },
    resetDrawingBox () {
      this.drawingBox.width = 0
      this.drawingBox.height = 0
      this.drawingBox.left = 0
      this.drawingBox.top = 0
      this.drawingBox.active = false
    },
    async saveImage () {
      // Validate all content name first
      if (this.validateAllContents()) {
        var labelPutPayload = []
        var labelPayload = []
        var tempArray = []
        var boxPosition 
        var bWidth 
        var bHeight 
        var realImageAttr 
        var content_id 
        var singleBackendObj 
        var move = 0
        var countBox = Object.keys(this.boxes).length
        var imageAttributes = {
          screenWidth: this.$refs.image.clientWidth,
          screenHeight: this.$refs.image.clientHeight,
          realWidth: this.$refs.image.naturalWidth,
          realHeight: this.$refs.image.naturalHeight
        }
        const imagePosition = this.getImagePositionRelativeToScreen(imageAttributes)
        if(this.boxesCount > countBox) {
          for (let delKey in this.deletedBoxes) {
            if (delKey == Object.keys(this.deletedBoxKey)[delKey - 1 - move] ) {
              try {
                await this.deleteLabelsInImage(Object.values(this.deletedBoxKey)[delKey - 1 - move])
              } catch (error) {
                this.showFailedAlert("Error!", error)
                return
              }
              for (let prevKey in this.previouslyCreatedBox) {
                if(Object.keys(this.previouslyCreatedBox)[prevKey] == Object.keys(this.deletedBoxKey)[delKey]) {
                  delete this.previouslyCreatedBox[Object.keys(this.deletedBoxKey)[prevKey]]
                }
              }
            }
          }  
        } 
        for (let key in this.boxes) {
          if (key == Object.keys(this.previouslyCreatedBox)[key - 1 - move] ) {
            // edit
            boxPosition = this.getBoxPositionRelativeToImage(imagePosition, key, this.boxes)
            bWidth = this.boxes[key].width
            bHeight = this.boxes[key].height
            realImageAttr = this.getRealImageAttributes(imageAttributes, boxPosition, bWidth, bHeight)
            try {
              content_id = await this.createLabelContent(this.boxes[key].content)
              singleBackendObj = {
                // TODO: change temporary image_id of 1 to real image_id
                image_id: parseInt(this.image.id),
                label_x_center: realImageAttr.xCenter,
                label_y_center: realImageAttr.yCenter,
                label_width: realImageAttr.width,
                label_height: realImageAttr.height,
                label_content_id: content_id
              }
              labelPutPayload.push(singleBackendObj)
              await this.changeLabelsInImage(labelPutPayload[0], Object.values(this.previouslyCreatedBox)[key - 1 - move])
            } catch (error) {
              this.showFailedAlert("Error!", error)
              return
            }
            delete this.previouslyCreatedBox[Object.keys(this.previouslyCreatedBox)[key - 1 - move]]
            move++
            labelPutPayload.pop()
          } else {
            tempArray.push(this.boxes[key])
          }
        }
        // push new label
        if (tempArray != 0) {
          for (let idxBox in tempArray) {
            boxPosition = this.getBoxPositionRelativeToImage(imagePosition, idxBox, tempArray)
            bWidth = tempArray[idxBox].width
            bHeight = tempArray[idxBox].height
            realImageAttr = this.getRealImageAttributes(imageAttributes, boxPosition, bWidth, bHeight)
            try {
              content_id = await this.createLabelContent(tempArray[idxBox].content)
              singleBackendObj = {
                image_id: parseInt(this.image.id),
                label_x_center: realImageAttr.xCenter,
                label_y_center: realImageAttr.yCenter,
                label_width: realImageAttr.width,
                label_height: realImageAttr.height,
                label_content_id: content_id
              }
              labelPayload.push(singleBackendObj)
              console.log('Label Payload: ', labelPayload)
            } catch (error) {
              this.showFailedAlert("Error!", error)
              return
            }
          }
          try {
            await this.createAllLabelsInImage(labelPayload)
            this.showSuccessAlert("Success!", "Image has been saved!").then(() => {
              this.closeViewer()
            })
          } catch (error) {
            console.error(error)
            this.showFailedAlert("Error!", error)
            return
          }
        } else {
          this.showSuccessAlert("Success!", "Image has been saved!").then(() => {
            this.closeViewer()
          })
        }
      }
    },
    validateAllContents () {
      for (let idxBox in this.boxes) {
        if (this.boxes[idxBox]) {
          if (this.boxes[idxBox].content === '' || !this.boxes[idxBox].content) {
            this.showFailedAlert("Content cannot be empty!", "Please check box with id = " + idxBox)
            return false
          }
        }
      }
      return true
    },
    showFailedAlert (title, bodyText) {
      return this.$swal.fire({
        title: title,
        text: bodyText,
        icon: 'error'
      })
    },
    showSuccessAlert(title, bodyText) {
      return this.$swal.fire({
        title: title,
        text: bodyText,
        icon: 'success'
      })
    },
    getImagePositionRelativeToScreen (imageAttributes) {
      const wrapperHeight = this.$refs.imageWrapper.clientHeight
      const wrapperWidth = this.$refs.imageWrapper.clientWidth
      const imageLeft = (wrapperWidth - imageAttributes.screenWidth) / 2
      const imageTop = (wrapperHeight - imageAttributes.screenHeight) / 2
      return {
        imageLeft: imageLeft,
        imageTop: imageTop
      }
    },
    getBoxPositionRelativeToImage (imagePosition, idxBox, boxes) {
      var bLeft = boxes[idxBox].left - imagePosition.imageLeft
      var bTop = boxes[idxBox].top - imagePosition.imageTop
      if (bLeft < 0) {
        bLeft = 0
      }
      if (bTop < 0) {
        bTop = 0
      }
      return {
        left: bLeft,
        top: bTop
      }
    },
    getScreenAttributes (imageAttributes, realBoxWidth, realBoxHeight, xCenterPosition, yCenterPosition) {
      const widthRatio = (imageAttributes.realWidth / imageAttributes.screenWidth)
      const heightRatio = (imageAttributes.realHeight / imageAttributes.screenHeight)
      const imagePosition = this.getImagePositionRelativeToScreen(imageAttributes)
      var bWidth = realBoxWidth / widthRatio
      var bHeight = realBoxHeight / heightRatio
      var bLeft = (xCenterPosition - (realBoxWidth / 2)) / widthRatio + imagePosition.imageLeft
      var bTop = (yCenterPosition - (realBoxHeight / 2)) / widthRatio + imagePosition.imageTop
      return {
        width: bWidth,
        height: bHeight,
        left: bLeft,
        top: bTop
      }
    },
    getRealImageAttributes (imageAttributes, boxPosition, bWidth, bHeight) {
      const widthRatio = (imageAttributes.realWidth / imageAttributes.screenWidth)
      const heightRatio = (imageAttributes.realHeight / imageAttributes.screenHeight)
      var realBoxWidth = widthRatio * bWidth
      var realBoxHeight = heightRatio * bHeight
      var xCenterPosition = widthRatio * boxPosition.left + (realBoxWidth / 2)
      var yCenterPosition = heightRatio * boxPosition.top + (realBoxHeight / 2)
      return {
        xCenter: xCenterPosition,
        yCenter: yCenterPosition,
        width: realBoxWidth,
        height: realBoxHeight
      }
    },
    async createLabelContent (content) {
      var url = '/api/content'
      var payload = {
        content_name: content
      }
      try {
        var response = await this.$axios.post(url, payload)
        var content_id = response.data.data['label_contents_id']
        return content_id
      } catch (error) {
        console.error(error)
        throw error
      }
    },
    async createAllLabelsInImage (labelPayload) {
      var url = '/api/label/many'
      try {
        var response = await this.$axios.post(url, labelPayload)
        return response.data.status
      } catch (error) {
        console.error(error)
        throw error
      }
    },
    async changeLabelsInImage (labelPayload, id) {
      var url ='api/label/' + id
      try {
        var response = await this.$axios.put(url, labelPayload)
        return response.data.status
      } catch (error) {
        console.error(error)
        throw error
      }
    },
    async deleteLabelsInImage (id) {
      var url ='api/label/' + id
      
      try {
        var response = await this.$axios.delete(url)
        return response.data.status
      } catch (error) {
        console.error(error)
        throw error
      }
    }
  }
}

</script>

<style scoped>
  .image {
     max-width: 85vw;
     max-height: 85vh;
     border-width: thin;
     cursor: cell;
  }

  .viewer-wrapper {
    /* height: 87.5vh; */
    width: 100vw;
  }

  .viewer-background {
    height: 100vh;
    background-color: #262626;
  }

  .center-horizontal {
    justify-content: center;
  }

  .btn-save-section {
    text-align: right;
    margin-top: -40px;
    margin-right: 5px;
  }

  .btn-text { 
    font-size: 0.85rem;
    /* font-family: 'Open Sans Bold'; */
  }

  .btn-close-text {
    font-size: 1.2rem;
  }

  .btn-close-section {
    text-align: right;
    margin-right: 1px;
    margin-top: -7px;
  }

  .btn-label-no-border {
    border: 0;
    color: white;
    background-color: #474747;
  }

  .btn-label-no-border:hover {
    background-color: #8e8383;
  }

  .btn-lg {
    width: 115px;
  }

  .flex-viewer {
    display: flex;
    height: 100vh;
  }
  
  .button-block {
    display: block;
    height: 100vh;
  }

</style>