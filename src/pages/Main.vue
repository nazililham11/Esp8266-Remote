<template>
	<div class="container-fluid">
		
		<!-- Connection -->
		<div class="row justify-content-center">
			<div class="col-12 my-3 connection-card-animated" :class="{ 'connection-card-animated-active': !is_connected }">
				<div class="card shadow h-100 py-2" :class="{
					'border-left-success': is_connected,
					'border-left-primary': !is_connected
					}">
					<div class="card-body">
						<div class="d-flex justify-content-between">
							<h5 class="m-0">
								<b :class="{
									'text-success': is_connected, 
									'text-primary': !is_connected
									}">Connection</b>
							</h5>
							<div class="spinner-border text-primary" role="status" v-if="is_loading"></div>
						</div>
						<hr>
						<div class="d-flex align-items-center">
							<transition name="width">
								<div class="input-group mr-4 flex-nowrap" v-if="is_connected">
									<div class="input-group-prepend">
										<label class="input-group-text" for="update-speed">Update speed</label>
									</div>
									<select class="custom-select" id="update-speed" v-model="update_speed">
										<option 
											v-for="(speed, index) in update_speeds"
											:key="index"
											:value="speed.value">
											{{ speed.label }}
										</option>
									</select>
								</div>
							</transition>

							<div class="input-group">
								<div class="input-group-prepend">
									<label for="ip-address" class="input-group-text text-nowrap">IP Address</label>
								</div>

								<input type="text" id="ip-address" class="form-control" v-model="ip_address" autofocus
									placeholder="IP Address" v-on:keyup.enter="toggle_connection" :disabled="is_connected">

								<div class="input-group-append">
									<button @click="toggle_connection" class="btn" :class="{
										'btn-success': !is_connected,
										'btn-danger': is_connected
									}" :disabled="ip_address.length < 7 || is_loading">{{ !is_connected ? 'Connect':'Disconnect' }}</button>
								</div>
							</div>
						</div>
					</div>
				</div>
			</div>
		</div>

		<transition name="swipe-down">
			<div class="row" v-if="is_connected">

				<!-- Wifi Config -->
				<div class="col-md-6 col-12 my-3">
					<div class="card border-left-success shadow h-100 py-2">
						<div class="card-body">
							<h5 class="text-success">
								<b>WIFI Config</b>
							</h5>
							<hr>
							<table class="table table-borderless">
								<tr v-for="(row, index) in wifi_config_table" :key="index">
									<td>{{ row.label }}</td>
									<td>:</td>
									<td v-if="row.label == 'Status'">
										<b class="text-success">
											{{ wifi_config.status == 3 ? 'Connected':'Disconnect' }}
										</b>
									</td>
									<td v-else>{{ row.value }}</td>
								</tr>
							</table>
						</div>
					</div>
				</div>
				
				<!-- Robot Moves -->
				<div class="col-md-6 col-12 my-3">
					<div class="card border-left-primary shadow h-100 py-2">
						<div class="card-body">
							<h5 class="text-primary"><b>Robot Moves</b></h5>
							<hr>
							<div class="d-flex flex-row">
								<b class="flex-fill">Basic</b>
								<button class="btn btn-outline-primary mx-3" @click="send_moves(1)">Stand</button>
								<button class="btn btn-outline-primary"	@click="send_moves(2)">Sit</button>
							</div>
							<hr>
							<div class="d-flex flex-row">
								<div class="d-flex flex-column">
									<b>Move</b>
									<input type="number" class="form-control" placeholder="Step" v-model="move_step">
								</div>
								<div class="d-flex flex-row flex-wrap justify-content-end">
									<button class="btn btn-outline-primary mx-1 mb-2" :disabled="move_step < 1" @click="send_moves(3)">
										Forward</button>
									<button class="btn btn-outline-primary mx-1 mb-2" :disabled="move_step < 1" @click="send_moves(4)">
										Back</button>
									<button class="btn btn-outline-primary mx-1 mb-2" :disabled="move_step < 1" @click="send_moves(5)">Turn
										Left</button>
									<button class="btn btn-outline-primary mx-1 mb-2" :disabled="move_step < 1" @click="send_moves(6)">Turn
										Right</button>
									<button class="btn btn-outline-primary mx-1 mb-2" :disabled="move_step < 1" @click="send_moves(7)">Body
										Left</button>
									<button class="btn btn-outline-primary mx-1 mb-2" :disabled="move_step < 1" @click="send_moves(8)">Body
										Right</button>
									<button class="btn btn-outline-primary mx-1 mb-2" :disabled="move_step < 1" @click="send_moves(9)">Hand
										Wave</button>
									<button class="btn btn-outline-primary mx-1 mb-2" :disabled="move_step < 1" @click="send_moves(10)">Hand
										Shake</button>
								</div>
							</div>

						</div>
					</div>
				</div>

			</div>
		</transition>


		<transition name="swipe-down">	
			<div class="row align-items-start" v-if="is_connected">

				<!-- Robot Speed -->
				<div class="col-md-6 col-12 my-3">
					<div class="card border-left-primary shadow h-100 py-2">
						<div class="card-body">
							<h5 class="text-primary"><b>Robot Speeds</b></h5>
							<hr>
							<div class="d-flex justify-content-between align-items-center">
								<div class="d-flex flex-column flex-fill">
									<strong>Move speed multiple</strong>
									<p class="m-0">Multiple all speeds(default 1).</p>
								</div>
								<input type="number" class="form-control" placeholder="Speed" v-model="speeds.move_speed_multiple">
							</div>
							<hr>
							<div class="d-flex justify-content-between align-items-center">
								<div class="d-flex flex-column flex-fill">
									<strong>Spot turn speed</strong>
									<p class="m-0">Speed to turn left or right.</p>
								</div>
								<input type="number" class="form-control" placeholder="Speed" v-model="speeds.spot_turn_speed">
							</div>
							<hr>
							<div class="d-flex justify-content-between align-items-center">
								<div class="d-flex flex-column flex-fill">
									<strong>Leg move speed</strong>
									<p class="m-0">Speed to leg move.</p>
								</div>
								<input type="number" class="form-control" placeholder="Speed" v-model="speeds.leg_move_speed">
							</div>
							<hr>
							<div class="d-flex justify-content-between align-items-center">
								<div class="d-flex flex-column flex-fill">
									<strong>Body move speed</strong>
									<p class="m-0">Speed to move body.</p>
								</div>
								<input type="number" class="form-control" placeholder="Speed" v-model="speeds.body_move_speed">
							</div>
							<hr>
							<div class="d-flex justify-content-between align-items-center">
								<div class="d-flex flex-column flex-fill">
									<strong>Stand seat speed</strong>
									<p class="m-0">Speed to stand and seat.</p>
								</div>
								<input type="number" class="form-control" placeholder="Speed" v-model="speeds.stand_seat_speed">
							</div>

							<hr>
							<button type="submit" class="float-right btn btn-primary px-3" @click="send_speeds()">Save</button>

						</div>
					</div>
				</div>

				<!-- Legs Coordinate -->
				<div class="col-md-6 col-12 my-3">
					<div class="card border-left-info shadow h-100 py-2">
						<div class="card-body">
							<h5 class="text-info"><b>Leg Coordinate</b></h5>
							<hr>

							<div class="row justify-content-around">

								<div class="col-lg-3 col-6">
									<b class="text-center d-block">Front Left</b>
									<div class="input-group my-1">
										<div class="input-group-prepend">
											<label for="front-left-x" class="input-group-text">X</label>
										</div>
										<input type="number" class="form-control" id="front-left-x" v-model="leg_coordinates.front_left.x" placeholder="X">
									</div>
									<div class="input-group my-1">
										<div class="input-group-prepend">
											<label for="front-left-y" class="input-group-text">Y</label>
										</div>
										<input type="number" class="form-control" id="front-left-y" v-model="leg_coordinates.front_left.y" placeholder="Y">
									</div>
									<div class="input-group my-1">
										<div class="input-group-prepend">
											<label for="front-left-z" class="input-group-text">Z</label>
										</div>
										<input type="number" class="form-control" id="front-left-z" v-model="leg_coordinates.front_left.z" placeholder="Z">
									</div>
								</div>
								
								<div class="col-lg-3 col-6">
									<b class="text-center d-block">Front Right</b>
									<div class="input-group my-1">
										<div class="input-group-prepend">
											<label for="front-right-x" class="input-group-text">X</label>
										</div>
										<input type="number" class="form-control" id="front-right-x" v-model="leg_coordinates.front_right.x" placeholder="X">
									</div>
									<div class="input-group my-1">
										<div class="input-group-prepend">
											<label for="front-right-y" class="input-group-text">Y</label>
										</div>
										<input type="number" class="form-control" id="front-right-y" v-model="leg_coordinates.front_right.y" placeholder="Y">
									</div>
									<div class="input-group my-1">
										<div class="input-group-prepend">
											<label for="front-right-z" class="input-group-text">Z</label>
										</div>
										<input type="number" class="form-control" id="front-right-z" v-model="leg_coordinates.front_right.z" placeholder="Z">
									</div>
								</div>
								
								<div class="col-lg-3 col-6">
									<b class="text-center d-block">Rear Left</b>
									<div class="input-group my-1">
										<div class="input-group-prepend">
											<label for="rear-left-x" class="input-group-text">X</label>
										</div>
										<input type="number" class="form-control" id="rear-left-x" v-model="leg_coordinates.rear_left.x" placeholder="X">
									</div>
									<div class="input-group my-1">
										<div class="input-group-prepend">
											<label for="rear-left-y" class="input-group-text">Y</label>
										</div>
										<input type="number" class="form-control" id="rear-left-y" v-model="leg_coordinates.rear_left.y" placeholder="Y">
									</div>
									<div class="input-group my-1">
										<div class="input-group-prepend">
											<label for="rear-left-z" class="input-group-text">Z</label>
										</div>
										<input type="number" class="form-control" id="rear-left-z" v-model="leg_coordinates.rear_left.z" placeholder="Z">
									</div>
								</div>

								<div class="col-lg-3 col-6">
									<b class="text-center d-block">Rear Right</b>
									<div class="input-group my-1">
										<div class="input-group-prepend">
											<label for="rear-right-x" class="input-group-text">X</label>
										</div>
										<input type="number" class="form-control" id="rear-right-x" v-model="leg_coordinates.rear_right.x" placeholder="X">
									</div>
									<div class="input-group my-1">
										<div class="input-group-prepend">
											<label for="rear-right-y" class="input-group-text">Y</label>
										</div>
										<input type="number" class="form-control" id="rear-right-y" v-model="leg_coordinates.rear_right.y" placeholder="Y">
									</div>
									<div class="input-group my-1">
										<div class="input-group-prepend">
											<label for="rear-right-z" class="input-group-text">Z</label>
										</div>
										<input type="number" class="form-control" id="rear-right-z" v-model="leg_coordinates.rear_right.z" placeholder="Z">
									</div>
								</div>
							</div>
							<hr>
							<button type="submit" class="float-right btn btn-primary px-3">Save</button>
						</div>
					</div>
				</div>

			</div>
		</transition>

	</div>
</template>

<script>
	import axios from 'axios'

	export default {
		name: 'Main',
		props: { },
		data() {
			return {
				ip_address: '192.168.0.100',
				is_connected: false,
				is_loading: false,
				update_speed: 0,
				move_step: 1,

				wifi_config: {
					ssid: '-',
					ip_address: '-',
					status: '-',
					signal: '-',
					pass: '-',
				},
				speeds: {
					move_speed_multiple: 0, 
					spot_turn_speed: 0, 
					leg_move_speed: 0, 
					body_move_speed: 0, 
					stand_seat_speed: 0, 
				},
				update_speeds: [
					{ value: 100, 	label: '0.1s' }, 
					{ value: 250, 	label: '0.25s' }, 
					{ value: 500, 	label: '0.5s' }, 
					{ value: 1000, 	label: '1s' }, 
					{ value: 2000, 	label: '2s' }, 
					{ value: 0, 	label: 'Paused' }, 
				],
				leg_coordinates: {
					front_left:  {x: 0, y: 0, z: 0},
					front_right: {x: 0, y: 0, z: 0},
					rear_left: 	 {x: 0, y: 0, z: 0},
					rear_right:  {x: 0, y: 0, z: 0},
				},
				
				

			}
		},
		computed: {
			wifi_config_table(){
				return [
					{ label: 'Status', 		value: this.wifi_config.status },
					{ label: 'SSID', 		value: this.wifi_config.ssid },
					{ label: 'Password',	value: this.wifi_config.pass },
					{ label: 'IP Address', 	value: this.wifi_config.ip_address},
					{ label: 'Signal', 		value: this.wifi_config.signal },
				]
			},
			
			
		},
		methods: {
			url(param) {
				return 'http://' + this.ip_address + param
			},
			toggle_connection() {
				if (!this.is_connected)
					this.connect()
				else
					this.disconnect()
			},
			connect() {
				this.is_loading = true
				let vm = this

				axios.get(this.url('/connect'))
					.then(function (response) {
						vm.is_connected = true
						vm.wifi_config = response.data
						console.log(response)
					})
					.catch(function (error) {
						vm.is_connected = false
						console.log(error.config)
					})
					.then(function () {
						vm.is_loading = false
					})

			},
			disconnect() {
				this.is_connected = false
			},
			send_moves(move_id){
				this.is_loading = true
				let vm = this

				axios.get(this.url('/moves'), { params: { move_id: move_id, step: vm.move_step} })
					.then(function (response) {
						console.log(response)
					})
					.catch(function (error) {
						console.log(error.config)
					})
					.then(function () {
						vm.is_loading = false
					})
			},
			send_speeds(){
				this.is_loading = true
				let vm = this

				axios
					.get(this.url('/speeds'), { params: this.speeds })
					.then(function (response) {
						console.log(response)
					})
					.catch(function (error) {
						console.log(error.config)
					})
					.then(function () {
						vm.is_loading = false
					})
			},
			send_leg_coordinates(){
				this.is_loading = true
				let vm = this
				let params = {
					front_left_x: this.leg_coordinates.front_left.x,
					front_left_y: this.leg_coordinates.front_left.y,
					front_left_z: this.leg_coordinates.front_left.z,

					front_right_x: this.leg_coordinates.front_right.x,
					front_right_y: this.leg_coordinates.front_right.y,
					front_right_z: this.leg_coordinates.front_right.z,

					rear_left_x: this.leg_coordinates.rear_left.x,
					rear_left_y: this.leg_coordinates.rear_left.y,
					rear_left_z: this.leg_coordinates.rear_left.z,

					rear_right_x: this.leg_coordinates.rear_right.x,
					rear_right_y: this.leg_coordinates.rear_right.y,
					rear_right_z: this.leg_coordinates.rear_right.z,
				}
				axios
					.get(this.url('/leg_coordinates'), { params: params })
					.then(function (response) {
						console.log(response)
					})
					.catch(function (error) {
						console.log(error.config)
					})
					.then(function () {
						vm.is_loading = false
					})
			},
			
		}
	}
</script>

<style scoped>
	.table tr {
		height: 2rem;
		vertical-align: middle;
	}

	.table td {
		padding: 0 1rem;
	}

	.table tr td:first-child {
		white-space: nowrap;
		padding: 0;
	}

	.table tr td:last-child {
		width: 100%;
		padding: 0;
	}
	vr {
		height: 100%;
		width: 2px;
		border-right: 5px solid black;
	}
	input[type=number]{
		max-width: 7rem;
	}
	#leg_movement_configuration .input-group .input-group-prepend .input-group-text {
		min-width: 6rem;
	}
	/* Animation */

	/* Swipe Down */
	.swipe-down-enter-active, .swipe-down-leave-active {
		transition: all .5s;
	}
	.swipe-down-enter, .swipe-down-leave-to /* .fade-leave-active below version 2.1.8 */ {
		opacity: 0;
		margin-top: -5rem;
	}

	/* Width */
	.width-enter-active, .width-leave-active {
		transition: all 0.5s;
	}
	.width-enter, .width-leave-to {
		width: 0%;
		opacity: 0;
	}
	.width-enter-to, .width-leave {
		width: 100%;
		opacity: 1;
	}

	/* Connection Card Top */
	.connection-card-animated {
		transition: all 0.5s ease-in-out;
	}
	.connection-card-animated-active {
		transition: all 0.5s ease-in-out;
		flex-basis: 70%; 
		margin-top: 15% !important;
	}
</style>