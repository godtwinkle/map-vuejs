<template>
  <section class="ui two column centered grid">
    <div class="column">
      <form class="ui segment large form">
        <div class="ui message red"></div>
        <div class="ui-segment">
          <div class="field">
            <div class="ui right icon input large">
              <input type="text" placeholder="Enter your address" />
              <i
                class="dot circle link icon"
                @click="locatorButtonPressed()"
              ></i>
            </div>
          </div>
          <button class="ui button">Go</button>
        </div>
      </form>
    </div>
  </section>
</template>

<script>
export default {
  methods: {
    locatorButtonPressed() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
          (position) => {
            console.log(position.coords.latitude);
            console.log(position.coords.longitude);
          },
          (error) => {
            console.log(error.message);
          }
        );
      } else {
        console.log("Trình duyệt của bạn không hỗ trợ API này");
      }
    },
  },
};
</script>

<style>
.ui.button,
.dot.circle.icon {
  background-color: #ff5a5f;
  color: white;
}
</style>
