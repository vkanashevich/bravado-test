<template>
  <div class="container">
    <custom-input />
    <div class="pooch-container">
      <pooch class="-mt-20" name="Mr Cat" email="lollolo" />
      <pooch class="-mt-20" name="Ms Kitty" address="2220 moon solar system" position="lol kek cheburek" email="mooth@google.com" />
    </div>
  </div>
</template>

<script>
export default {}
</script>

<style lang="scss" scoped>
.container {
  max-width: 564px;
  margin: 0 auto;
  height: 100%;
  overflow: hidden;
  background: #fff;
  padding: 19px 12px 0;
}

.poach + .poach {
  margin-bottom: 20px;
}

</style>
