<template>
	<div id="app">
		<yandex-map ref="map" :coords="[long, lat]" @click="onClickMap" :controls="controls" :zoom="zoom" class="map" :settings="settings">
			<ymap-marker
				v-for="n in markers"
				:key="n.id"
				:marker-id="n.id"
				marker-type="placemark"
				:coords="n.coord"
				:hint-content="`Координаты: ${n.coord}, радиус: ${n.radius}`"
				@click="onClickPoint(n)"
			></ymap-marker>
			<ymap-marker
				v-for="n in markers"
				:key="n.id"
				:marker-id="n.id"
				marker-type="circle"
				:coords="n.coord"
				:circle-radius="n.radius"
				@click="onClickPoint(n)"
			></ymap-marker>
		</yandex-map>
		<div class="sidebar" v-if="markers.length" id="scroll">
			<div v-for="n in markers" :key="n.id">
				<div class="card blink" :class="{ active: n.selected }" :id="n.id">
					<div class="info">
						<div @click="onClickPoint(n)" class="inputs">
							<p>
								Координаты: <input type="text" :placeholder="n.coord[0]" v-model="n.coord[0]" /><input
									type="text"
									:placeholder="n.coord[1]"
									v-model="n.coord[1]"
									class="lat"
								/>
							</p>
							<p>Радиус (м): <input type="text" :placeholder="n.radius" v-model="n.radius" /></p>
						</div>

						<Slider @click="onClickSlider(m)" v-model="n.radius" :max="20000" class="slider" :step="10" />
						<div @click="onClickPoint(n)">
							<button @click.stop="onDeleteMark(n.id)" class="red">Удалить</button>
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
import { yandexMap, ymapMarker, loadYmap } from "vue-yandex-maps";
import Slider from "@vueform/slider";

export default {
	name: "App",
	components: {
		yandexMap,
		ymapMarker,
		Slider,
	},

	data() {
		return {
			markers: [
				//Тестовые точки
				// { coord: [53.139974192679674, 102.94056083902736], radius: 8000, selected: false },
				// { coord: [53.16061539709935, 103.32301970466995], radius: 9000, selected: false },
				// { coord: [53.17299436751333, 103.58257170662306], radius: 5000, selected: false },
				// { coord: [52.9952193919426, 103.16371794685745], radius: 11000, selected: false },
				// { coord: [53.020897593400555, 103.66908904060745], radius: 8000, selected: false },
			].map((n, i) => ({ ...n, id: i + 1 })),
			controls: ["rulerControl", "zoomControl", "searchControl", "geolocationControl", "typeSelector", "fullscreenControl"],
			zoom: 11,
			settings: {
				apiKey: "6dd8d184-cc60-4cdf-a5c2-1a419a87dbe2",
				lang: "ru_RU",
				coordorder: "latlong",
				enterprise: false,
				version: "2.1",
			},
			map: null,
			long: null,
			lat: null,
			active: false,
		};
	},

	async mounted() {
		await loadYmap();
		var geolocation = ymaps.geolocation;
		geolocation
			.get({
				provider: "auto",
			})
			.then((result) => {
				const [long, lat] = result.geoObjects.position;
				this.long = long;
				this.lat = lat;

				//На случай, если понадобится отображение текущего положения
				//  result.geoObjects.options.set('preset', 'islands#redDotIcon');
				//  this.$refs.map.myMap.geoObjects.add(result.geoObjects);
			});
	},

	methods: {
		templateBalloon(mark) {
			return `
                <p><strong>Координаты</strong>: ${mark.coord}</p>
                <p><strong>Радиус</strong>: ${mark.radius}</p>
            `;
		},

		onClickMap(e) {
			this.markers = [...this.markers, { coord: e.get("coords"), radius: 4000, id: this.markers.length + 1, selected: false }];
			this.long = e.get("coords")[0];
			this.lat = e.get("coords")[1];
			setTimeout(() => {
				const element = document.getElementById("scroll");
				element.scrollTop = element.scrollHeight - element.clientHeight;
			}, 10);
		},

		onClickPoint(m) {
			this.$refs.map.myMap.balloon.open(m.coord, this.templateBalloon(m));
			setTimeout(() => {
				this.$refs.map.myMap.setCenter(m.coord, 10);
			}, 100);

			document.getElementById(m.id).scrollIntoView({ behavior: "smooth", block: "center", inline: "center" });

			this.markers = this.markers.map((el) => (el.selected == true ? { ...el, selected: false } : el));
			m.selected = true;

			this.long = m.coord[0];
			this.lat = m.coord[1];
		},

		onClickSlider(m) {
			this.long = m.coord[0];
			this.lat = m.coord[1];

			setTimeout(() => {
				this.$refs.map.myMap.setCenter(m.coord, 10);
			}, 100);
		},

		onDeleteMark(id) {
			this.markers = this.markers.filter((obj) => obj.id !== id);
			console.log(this.markers);
			this.$refs.map.myMap.balloon.close();
		},
	},
};
</script>

<style src="@vueform/slider/themes/default.css"></style>

<style>
#app {
	width: 100%;
	display: flex;
	flex-direction: row;
	height: 98vh;
}

.sidebar {
	display: flex;
	flex-direction: column;
	margin-left: 15px;
	overflow-y: scroll;
	overflow-x: hidden;
	font-size: 18px;
	max-width: 20%;
}

.map {
	width: 100%;
	overflow: hidden;
	height: 100%;
}

.lat {
	margin-top: 5px;
}

.list {
	display: flex;
	flex-direction: row;
}

input {
	font-size: 16px;
	line-height: 20px;
	padding: 8px 16px;
	width: 80%;
	min-height: 20px;
	border: unset;
	border-radius: 4px;
	outline-color: rgb(84 105 212 / 0.5);
	background-color: rgb(255, 255, 255);
}

.ymap-container {
	height: 100%;
	overflow: hidden;
}

.card {
	display: flex;
	flex-direction: row;
	box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.1);
	background-color: #f1f1f1;
	margin: 5px;
	cursor: pointer;
	border-radius: 5px;
}

.inputs {
	margin-bottom: 45px;
	margin-left: 15px;
}

.active {
	display: flex;
	flex-direction: row;
	box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.1);
	margin: 5px;
	cursor: pointer;
	border-radius: 5px;
	background-color: #cef2fd;
}

button {
	color: #444444;
	background: #f3f3f3;
	border: 1px #dadada solid;
	padding: 10px 15px;
	border-radius: 5px;
	font-weight: bold;
	font-size: 9pt;
	outline: none;
	margin-top: 20px;
	margin-bottom: 10px;
	margin-left: 65%;
}

button:hover {
	border: 1px #c6c6c6 solid;
	box-shadow: 1px 1px 1px #eaeaea;
	color: #333333;
	background: #f7f7f7;
	cursor: pointer;
}

button:active {
	box-shadow: inset 1px 1px 1px #dfdfdf;
}

button.red {
	background: -webkit-linear-gradient(top, #dd4b39, #d14836);
	background: -moz-linear-gradient(top, #dd4b39, #d14836);
	background: -ms-linear-gradient(top, #dd4b39, #d14836);
	border: 1px solid #dd4b39;
	color: white;
	text-shadow: 0 1px 0 #c04131;
}

button.red:hover {
	background: -webkit-linear-gradient(top, #dd4b39, #c53727);
	background: -moz-linear-gradient(top, #dd4b39, #c53727);
	background: -ms-linear-gradient(top, #dd4b39, #c53727);
	border: 1px solid #af301f;
}

button.red:active {
	box-shadow: inset 0 1px 1px rgba(0, 0, 0, 0.2);
	background: -webkit-linear-gradient(top, #d74736, #ad2719);
	background: -moz-linear-gradient(top, #d74736, #ad2719);
	background: -ms-linear-gradient(top, #d74736, #ad2719);
}

.slider {
	width: 85%;
	margin-left: 15px;
}

@-webkit-keyframes blue-fade {
	from {
		background: #cef2fd;
	}
	to {
		background: #f1f1f1;
	}
}
@-moz-keyframes blue-fade {
	from {
		background: #cef2fd;
	}
	to {
		background: #f1f1f1;
	}
}
@keyframes blue-fade {
	from {
		background: #cef2fd;
	}
	to {
		background: #f1f1f1;
	}
}
.blink {
	-webkit-animation: blue-fade 2s ease-in-out 0s;
	-moz-animation: blue-fade 2s ease-in-out 0s;
	-o-animation: blue-fade 2s ease-in-out 0s;
	animation: blue-fade 2s ease-in-out 0s;
}

@media (width: 1024px) {
	.sidebar {
		max-width: 30%;
	}

	#app {
		height: 97vh;
	}
}
</style>
