<template>
    <div class="content-wrapper">
        <div class="row text-center align-items-center">
            <div class="col col-md-7 order-md-1 col-12 order-2 google-map-wrapper">
                <div id="google-map" :style="`width: ${mapDimensions.width}; height: ${mapDimensions.height}; position: relative; overflow: hidden;`"></div>
            </div>
            <div class="col col-md-5 order-md-2 col-12 mb-3 order-1">
                <div class="row justify-content-center p-3">
                    <div class="col col-10">
                        <h2>Weather Discovery Map</h2>
                        <p class="small">
                            Select a point on the map, or drag marker to a new position, to
                            find real-time data on the local weather
                        </p>
                    </div>
                </div>
                <template v-if="weather">
                    <div class="row justify-content-center mx-3" v-for="(item, key) of weather" :key="key">
                        <div class="col col-md-5 col-6 my-2">
                            <span class="badge badge-primary w-100 px-3 py-2">{{ key }}</span>
                        </div>
                        <div class="col col-md-7 col-6 my-2 weather-data">
                            {{ loading ? '---' : item }}
                        </div>
                    </div>
                </template>
            </div>
        </div>
    </div>
</template>

<script>
import { Loader } from "@googlemaps/js-api-loader";
import axios from "axios";

export default {
  data() {
    return {
        googleApiKey: "AIzaSyB41Bg4t97fmYwKs3MEL9DmicL53xQKpAY",
        weatherApiKey: "6d78fcf2b6ddf4f00ae680a37639b3d6",
        latlng: {
            lat: 40.76575889443308,
            lng: -111.89062013069112,
        },
        map: null,
        mapDimensions: {
            width: '58.333vw',
            height: '100vh'
        },
        windowWidth: 0,
        marker: null,
        loading: false,
        weather: null,
    };
  },

  mounted() {
    this.onResize();
    window.addEventListener("resize", this.onResize);

    var mapLoader = new Loader({
        apiKey: this.googleApiKey,
        version: "weekly",
        libraries: ["places"],
    })
    .load()
    .catch((err) => {
        console.warn("Err loading map: ", err);
    })
    .finally(() => {
        this.loadGoogleMap();
    });
  },

  methods: {
    onResize() {
        this.windowWidth = window.innerWidth;

        if(this.windowWidth <= 768) {
            this.mapDimensions.width = '90vw';
            this.mapDimensions.height = '35vh';
        } else {
            this.mapDimensions.width = '58.333vw';
            this.mapDimensions.height = '100vh';
        }
    },
    loadGoogleMap() {
        this.map = new google.maps.Map(document.getElementById("google-map"), {
            center: this.latlng,
            zoom: 10,
            gestureHandling: "none",
        });
        this.setMarker(this.latlng);
        google.maps.event.addListener(this.map, "click", (e) => {
            this.latlng = {
            lat: e.latLng.lat(),
            lng: e.latLng.lng(),
            };
            this.setMarker(e.latLng);
            this.map.setCenter(this.latlng);
        });
    },
    setMarker(location) {
        if (this.marker) {
            this.marker.setPosition(location);
        } else {
            this.marker = new google.maps.Marker({
            position: location,
            map: this.map,
            draggable: true,
            });
            google.maps.event.addListener(this.marker, "dragend", (data) => {
            this.latlng = {
                lat: this.marker.getPosition().lat(),
                lng: this.marker.getPosition().lng(),
            };
            this.map.setCenter(this.latlng);
            this.getWeatherData();
            });
        }

        this.getWeatherData();
    },
    getWeatherData() {
        this.loading = true;
        axios.get(`http://api.openweathermap.org/data/2.5/weather?lat=${this.latlng.lat}&lon=${this.latlng.lng}&units=imperial&appid=${this.weatherApiKey}`)
        .then((res) => {
            this.weather = {
                Location: res.data.name,
                Condition: res.data.weather[0].main,
                Temperature: `${res.data.main.temp} F`,
                Humidity: `${res.data.main.humidity}%`,
                Sunrise: this.toDateString(res.data.sys.sunrise),
                Sunset: this.toDateString(res.data.sys.sunset),
                "Wind Speed": `${res.data.wind.speed} mph`,
                "Wind Direction": `${res.data.wind.deg} degrees`,
            };
        })
        .catch((err) => {
            console.warn("Error fetching weather data: ", err);
        })
        .finally(() => {
            this.loading = false;
        });
    },
    toDateString(unixString) {
        return new Date(unixString * 1000).toLocaleString("en-US", {
            hour: "numeric",
            minute: "numeric",
            timeZone: "America/Denver",
            timeZoneName: "short",
        });
    },
  },
};
</script>

<style lang="scss">
.content-wrapper {
    font-size: 20px;

    & .weather-data {
        text-align: left;
        background-color: #00000008;
        border-radius: 0.25rem;
    }

    & .google-map-wrapper {
        text-align: center;
        text-align: -webkit-center;
    }
}

</style>
