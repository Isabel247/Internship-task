<template>
  <VPerfectSignature height="200px" width="720px"
    :stroke-options="strokeOptions" ref="signaturePad" />
  <div class="q-pa-xs q-gutter-sm ">
    <q-btn color="red" label="Undo" @click="undo"></q-btn>
    <q-btn color="primary" label="Save" @click="save"></q-btn>
  </div>
</template>

<script>
import { ref } from 'vue';
import VPerfectSignature from 'v-perfect-signature';

export default ({
  name: 'ConsentSignature',
  components: {
    VPerfectSignature,
  },
  setup(props, context) {
    const signaturePad = ref(null);
    const strokeOptions = {
      size: 2,
      thinning: 0.5,
      smoothing: 0.5,
      streamline: 0.5,
    };

    const save = () => {
      if (signaturePad.value.isEmpty) {
        const data = signaturePad.value.toDataURL();
        context.emit('onSigned', data);
      }
    };

    const undo = () => {
      signaturePad.value.clear();
      console.log('clear');
    };
    return {
      signaturePad, strokeOptions, save, undo,
    };
  },

});
</script>

<style scooped>
canvas{
  border: 1px solid black;
  height: 300px;
  width:  500px;
  cursor: crosshair;
}

</style>
