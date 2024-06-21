<template>
  <div v-if="visible" class="dialog-overlay" @click="close">
    <div :class="['dialog-content', dialogClass]" @click.stop>
      <header class="dialog-header">
        <slot name="header">
          <h3>{{ title }}</h3>
        </slot>
      </header>
      <section class="dialog-body">
        <slot>
          Default body content
        </slot>
      </section>
      <footer class="dialog-footer">
        <slot name="footer">
          <button @click="close">Close</button>
        </slot>
      </footer>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    title: {
      type: String,
      default: 'Dialog Title'
    },
    visible: {
      type: Boolean,
      default: false
    },
    type: {
      type: String,
      default: 'default'
    }
  },
  methods: {
    close() {
      this.$emit('update:visible', false);
    }
  }
};
</script>

<style scoped>
.dialog-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.dialog-content {
  font-family: 'Courier New', Courier, monospace;
  font-weight: bolder;
  color:black;
  background: white;
  padding: 20px;
  border-radius: 1vmin;
  max-width: 100vmin;
  width: 100%;
}
.dialog-content p{
  font-size: 3vmin;
}

.dialog-header,
.dialog-body,
.dialog-footer {
  margin-bottom: 1vmin;
}
.dialog-header{
  font-size: 4vmin;
}
.dialog-footer{
  align-items: center; 
  font-size: 1.5vmin;
} 


.dialog-footer button{
  text-align: center;
  width: 10vw;
  height: 4vh;
  z-index: 999;
  font-weight: bold;
  color:#ccc;
  padding: 1vmin;
  outline: none;
  border: none;
  background-color: rgb(4, 133, 138);

}
.dialog-footer button:hover{
  cursor: pointer;
  animation: jelly 1s;
  color:aliceblue;
}
@keyframes jelly {
  0%,
  100%{
    transform: scale(1,1);
  }
  25%{
    transform: scale(0.9,1.1);
  }
  55%{
    transform: scale(1.1,0.9);
  }
  75%{
    transform: scale(0.95,1.05);
  }

}

</style>
