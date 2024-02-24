<template>
  <div class="ui grid">
    <div class="six wide column red">
      <form class="ui segment large form">
        <div class="ui message red" v-show="error">{{ error }}</div>
        <div class="ui segment">
          <div class="field">
            <div
              class="ui right icon input large"
              :class="{ loading: spinner }"
            >
              <input
                type="text"
                placeholder="Enter your address"
                v-model="address"
                ref="autocomplete"
              />
              <i class="dot circle link icon" @click="locatorButtonPressed"></i>
            </div>
          </div>

          <div class="field">
            <div class="two fields">
              <div class="field">
                <select v-model="type">
                  <option value="restaurant">Restaurant</option>
                </select>
              </div>

              <div class="field">
                <select v-model="radius">
                  <option value="5">5KM</option>
                  <option value="10">10KM</option>
                  <option value="15">15KM</option>
                  <option value="20">20KM</option>
                </select>
              </div>
            </div>
          </div>
          <button class="ui button" @click="findCloseBuyButtonPressed">
            Find CloseBuy Places
          </button>
        </div>
      </form>

      <div class="ui segment" style="max-width: 300px; overflow: auto">
        <div class="ui divided items">
          <div
            class="item"
            v-for="(place, index) in places"
            :key="place.id"
            @click="showInfoWindow(index)"
            :class="{ active: activeIndex === index }"
            style="padding: 10px"
          >
            <div class="content">
              <div class="header">{{ place.name }}</div>
              <div class="meta">{{ place.vicinity }}</div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="ten wide column" ref="map"></div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      address: "",
      error: "",
      spinner: false,
      apiKey: "",
      lat: 0,
      lng: 0,
      type: "",
      radius: "",
      place: [],
      markers: [],
      activeIndex: -1,
    };
  },

  mounted() {
    var autocomplete = new google.maps.places.Autocomplete(
      this.$refs["autocomplete"],
      {
        bounds: new google.maps.LatLngBounds(
          new google.maps.LatLng(45.4215296, -75.6971931)
        ),
      }
    );

    autocomplete.addListener("place_changed", () => {
      var place = autocomplete.getPlace();

      this.showLocationOnTheMap(
        place.geometry.location.lat(),
        place.geometry.location.lng()
      );
    });
  },

  methods: {
    locatorButtonPressed() {
      this.spinner = true;

      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
          (position) => {
            this.lat = position.coords.latitude;
            this.lng = position.coords.longitude;

            this.getAddressFrom(
              position.coords.latitude,
              position.coords.longitude
            );

            this.showLocationOnTheMap(
              position.coords.latitude,
              position.coords.longitude
            );
          },
          (error) => {
            this.error = error.message;
            // "Locator is unable to find your address. Please type your address manually.";
            this.spinner = false;
            // console.log(error.message);
          }
        );
      } else {
        this.error =
          "Định vị không thể tìm thấy địa chỉ của bạn. Vui lòng nhập địa chỉ";
        this.spinner = false;
        // console.log("Your browser does not support geolocation API ");
        console.log("Trình duyệt của bạn không hỗ trợ định vị");
      }
    },
    getAddressFrom(lat, long) {
      axios
        .get(
          "https://maps.googleapis.com/maps/api/geocode/json?latlng=" +
            lat +
            "," +
            long +
            "&key=" +
            this.apiKey
        )
        .then((response) => {
          if (response.data.error_message) {
            this.error = response.data.error_message;
            console.log(response.data.error_message);
          } else {
            this.address = response.data.results[0].formatted_address;
            // console.log(response.data.results[0].formatted_address);
          }
          this.spinner = false;
        })
        .catch((error) => {
          this.error = error.message;
          this.spinner = false;
          console.log(error.message);
        });
    },

    showLocationOnTheMap(latitude, longitude) {
      // Show & center the Map based oon
      var map = new google.maps.Map(this.$refs["map"], {
        zoom: 15,
        center: new google.maps.LatLng(latitude, longitude),
        mapTypeId: google.maps.MapTypeId.ROADMAP,
      });

      new google.maps.Marker({
        position: new google.maps.LatLng(latitude, longitude),
        map: map,
      });
    },

    findCloseBuyButtonPressed() {
      const URL = `http://map.googleapi.com/map/api/place/nearbysearch/json?location=
      ${this.lat},${this.lng}&type=${this.type}&radius=${
        this.radius * 1000
      }&key=${this.apiKey}`;

      axios
        .get(URL)
        .then((response) => {
          this.places = response.data.results;
          this.showPlacesOnMap();
          console.log(response);
        })
        .catch((error) => {
          this.error = error.message;
        });
    },

    showPlacesOnMap() {
      const map = new google.maps.Map(this.$refs["map"], {
        zoom: 15,
        center: new google.maps.LatLng(this.lat, this.lng),
        mapTypeId: google.maps.MapTypeId.ROADMAP,
      });

      const infoWindow = new google.maps.InfoWindow();

      for (let i = 0; i < this.places.length; i++) {
        const lat = this.places[i].geometry.location.lat;
        const lng = this.places[i].geometry.location.lng;
        const placeID = this.places[i].place_id;

        const marker = new google.map.Marker({
          position: new google.map.LatLng(lat, lng),
          map: map,
        });

        this.markers.push(marker);

        google.map.event.addListener(marker, "click", function () {
          const URL = `https://cors-anywhere.herokuapp.com/https://maps.googleapis.com/maps/api/place/details/json?key=${this.apiKey}&place_id=${placeID}`;

          axios
            .get(URL)
            .then((response) => {
              if (response.data.error_message) {
                this.error = response.data.error_message;
              } else {
                const place = response.data.result;

                infoWindow.setContent(
                  `<div class="ui header">${this.places[i].name}</div>
            ${place.formatted_address} <br>
            ${place.formatted_phone_number} <br>
            <a href="${place.website}" targer="_blank">${place.website}</a>`
                );
                infoWindow.open(map, marker);
              }
            })
            .catch((error) => {
              this.error = error.message;
            });
        });
      }
      new markerClusterer.MarkerClusterer({
        map,
        markers: this.markers,
      });
    },

    showInfoWindow(index) {
      this.activeIndex = index;
      new google.maps.event.trigger(this.markers[index], "click");
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

.pac-icon {
  display: none;
}

.pac-item {
  padding: 10px;
  font-size: 16px;
  cursor: pointer;
}

.pac-item:hover {
  background-color: #ececec;
}

.pac-item-query {
  font-size: 16px;
}

#map {
  position: absolute;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
}

.active {
  background-color: #ff5a5f !important;
}
</style>
