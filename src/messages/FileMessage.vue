<template>
    <div>
        <div class='sc-message--file' v-for="attachment in data.file" :style="messageColors">
            <div class='sc-message--file-icon' v-if="['image/gif','image/jpg','image/jpeg','image/png','image/tiff','image/svg'].includes(attachment.mime_type)">
                <a :href="attachment.url ? attachment.url : '#'" target='_blank'><img :src="attachment.url" class="sc-image"></a>
            </div>
            <div class='sc-message--file-name' v-else :style="messageColors">
                <a :href="attachment.url ? attachment.url : '#'" target='_blank'>
                <span class='icon-file-message'><img :src="icons.file.img"  height="15px" style="height: 15px" /></span>
                Attachment</a>
            </div>
        </div>
    </div>
</template>

<script>
import fileIcon from '../assets/file.svg'
export default {
  props: {
    data: {
      type: Object,
      required: true
    },
    icons:{
      type: Object,
      required: false,
      default: function () {
        return {
          file:{
            img: fileIcon,
            name: 'default',
          },
        }
      }
    },
    messageColors: {
      type: Object,
      required: true
    }
  }
}
</script>

<style scoped>
.sc-message--file {
  border-radius: 6px;
  font-weight: 300;
  font-size: 14px;
  line-height: 1.4;
  /* white-space: pre-wrap; */
  -webkit-font-smoothing: subpixel-antialiased
}

.sc-message--content.sent .sc-message--file {
  word-wrap: break-word;
}

.sc-message--file-icon {
  text-align: center;
  margin-left: auto;
  margin-right: auto;
  margin-top: 15px;
  margin-bottom: 15px;
}

.sc-image {
  max-width: 100%;
  min-width: 100%;
}

.sc-message--file-text {
  padding: 17px 20px;
  border-radius: 6px;
  font-weight: 300;
  font-size: 14px;
  line-height: 1.4;
  white-space: pre-wrap;
  -webkit-font-smoothing: subpixel-antialiased
}

.sc-message--file-name {
  color: white;
  padding-top: 0;
  font-size: small;
  text-align: center;
}

.sc-message--file-name a {
  text-decoration: none;
  color: #ece7e7;
}

.sc-message--file-name a:hover {
  color: white;
}

.sc-message--content.sent .sc-message--file-text {
  color: white;
  background-color: #4e8cff;
  word-wrap: break-word;
}

.sc-message--content.received .sc-message--file {
  color: #263238;
  background-color: #f4f7f9;
}

.sc-message--content.received .sc-message--file-name {
  color: #000;
}

.sc-message--content.received .sc-message--file a {
  color: rgba(43, 40, 40, 0.7);
}

.sc-message--content.received .sc-message--file a:hover {
  color: #0c0c0c;
}
</style>