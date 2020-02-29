<template>
	<div class="container-fluid">
		
		<!-- Connection -->
		<div class="row">
			<div class="col-12 my-3">
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
							<div class="spinner-border text-primary" role="status" v-if="is_loading">
								<span class="sr-only">Loading...</span>
							</div>
						</div>
						<hr>
						<div class="d-flex flex-row align-items-center">
							<div class="input-group mr-4">
								<div class="input-group-prepend">
									<label class="input-group-text" for="update-speed">Update speed</label>
								</div>
								<select class="custom-select w-50" id="update-speed" :disabled="!is_connected">
									<option value="100">0.1s</option>
									<option value="250">0.25s</option>
									<option value="500">0.5s</option>
									<option value="1000">1s</option>
									<option value="2000">2s</option>
									<option value="0" selected>Paused</option>
								</select>
							</div>

							<div class="input-group col-md-8">
								<div class="input-group-prepend">
									<label for="ip-address" class="input-group-text text-nowrap">IP Address</label>
								</div>

								<input type="text" id="ip-address" class="form-control" v-model="ip_address"
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
							<tr>
								<td>Status</td>
								<td>:</td>
								<td><b class="text-success">{{ wifi_conf.status == 3 ? 'Connected':'' }}</b></td>
							</tr>
							<tr>
								<td>SSID</td>
								<td>:</td>
								<td>{{ wifi_conf.ssid }}</td>
							</tr>
							<tr>
								<td>Password</td>
								<td>:</td>
								<td>{{ wifi_conf.pass }}</td>
							</tr>
							<tr>
								<td>IP Address</td>
								<td>:</td>
								<td>{{ wifi_conf.ip_address }}</td>
							</tr>
							<tr>
								<td>Signal</td>
								<td>:</td>
								<td>{{ wifi_conf.signal }}</td>
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

	</div>
</template>

<script>
	import axios from 'axios'

	export default {
		name: 'Main',
		props: {
			msg: String
		},
		data() {
			return {
				ip_address: '192.168.0.103',
				is_connected: false,
				is_loading: false,
				move_step: 0,
				wifi_conf: {
					ssid: '',
					ip_address: '',
					status: '',
					signal: '',
					pass: '',
				}
			}
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
						vm.wifi_conf = response.data
						console.log(response);
					})
					.catch(function (error) {
						vm.is_connected = false
						console.log(error.config);
					})
					.then(function () {
						vm.is_loading = false
					});

			},
			disconnect() {
				this.is_connected = false
			},
			send_moves(move_id){
				this.is_loading = true
				let vm = this

				axios.get(this.url('/move'), { move_id: move_id, step: vm.move_step })
					.then(function (response) {
						console.log(response);
					})
					.catch(function (error) {
						console.log(error.config);
					})
					.then(function () {
						vm.is_loading = false
					});
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
</style>