<template>
	<b-container>

	  	<div style="display: inline-block;
    width: 100%;
    padding: 20px 0px;">
	  		  <b-button-toolbar key-nav class="pull-right"  aria-label="Toolbar">

			    <!--
			    <b-button-group  size="sm" class="mx-1">
					<b-dropdown size="sm"  class="mx-1" right text="Filtern">
				      <b-dropdown-item>Berlin</b-dropdown-item>
				      <b-dropdown-item>Lübeck</b-dropdown-item>
				    </b-dropdown>
			    </b-button-group>
			    -->

			    <b-button-group  size="sm" class="mx-1">
					<b-btn variant="primary" @click="openAddNew"><i class="fa fa-plus"></i>&nbsp; Event hinzufügen</b-btn>
			    </b-button-group>

			   </b-button-toolbar>
	  	</div>

	  		<!--
	  		<b-row class="py-3 px-2">

			  <b-col style="max-width: 60px;">
			  <h5>Filter: </h5>
			  </b-col>

			  <b-col>
			  <b-badge href="#" variant="primary">Filter <span class="dismiss">&times;</span> </b-badge>
			  </b-col>

			</b-row>
	  		-->
	  		<b-list-group class="event-list" v-if="this.events.length > 0">
			  <b-list-group-item class="flex-column align-items-start" v-for="event in this.events" v-bind:data="event" v-bind:key="event.id">

			    <b-row class="row-eq-height">

			        <!-- ev is in same month -->
			        <b-col lg="2" md="3" sm="12" class="etc-same-month" v-if="new Date(event.startDate).getMonth() == new Date(event.endDate).getMonth()">



			        	 <div v-if="new Date(event.startDate).getDay() == new Date(event.endDate).getDay()">

			        	 	<span class="date">

			        	 		<span class="day">{{ event.startDate | dateDay() }}.</span>
			        	 		<span class="month">{{ event.startDate | dateMonthName() }}</span>
			        	 		<span class="year">{{ event.startDate | dateYear() }}</span>

			        	 	</span>
			        	 </div>
			        	 <div v-if="new Date(event.startDate).getDay() != new Date(event.endDate).getDay()">
			        	 	<span class="date">
										<span class="day">{{ event.startDate | dateDay() }}. - {{ event.endDate | dateDay() }}.</span>
			        	 		<span class="month">{{ event.startDate | dateMonthName() }}</span>
			        	 		<span class="year">{{ event.startDate | dateYear() }}</span>
									</span><br/>
								 </div>

			        </b-col>
			        <!-- ev is not in same month -->
			        <b-col lg="2" md="3" sm="12" class="etc-not-same-month" v-if="new Date(event.startDate).getMonth() != new Date(event.endDate).getMonth()">
			        	{{ event.startDate | regularDate() }} bis {{ event.endDate | regularDate() }}
			        </b-col>

			        <b-col lg="10" md="9" sm="12">


			        	<span class="time">{{ event.startDate | dateTime() }} – {{ event.endDate | dateTime() }}</span>
						<span class="readable-time-left" v-if="( new Date(event.startDate).getTime() - (new Date().getTime()) ) < (7 * 24 * 60 * 60 * 1000)">
				        	 <small>in {{ (new Date(event.startDate).getTime() - (new Date().getTime() )) | readableTime() }}</small>
				         </span>

			        	<h3 class="mb-1">{{ event.title }}</h3>
			        	<h5>
				        	<small v-if="event.location"><i class="fa fa-building"></i> {{event.location}}</small>
				        	<small>{{event.brand}}</small>
				        	<small v-if="event.room"><i class="fa fa-tag"></i> {{event.room}}</small>
				        	<small v-if="event.organiser"><i class="fa fa-user"></i> {{event.organiser}}</small>

			        	</h5>

								<vue-markdown>{{event.description}}</vue-markdown>

			        </b-col>

			        <div class="event-actions">
				        <b-button size="sm" variant="link" @click="assignEdit(event)">
							<i class="fa fa-edit"></i> Event bearbeiten
			            </b-button>
			        </div>

			    </b-row>
			  </b-list-group-item>
			</b-list-group>

			  <div>
			  <!-- Modal Component -->
			  <b-modal id="modal1" ref="modal1" size="lg" v-bind:title="modalTitle()" cancel-title="Abbrechen" @ok="onSubmit" ok-title="Speichern">


			  	<b-alert show v-if="status == 'load'"><div class="loader"></div>&nbsp; <span style="position: relative;
    top: -5px;">Sende Daten ...</span></b-alert>

			    <b-alert show v-if="responsetxt != ''">{{responsetxt}}</b-alert>

			    <b-form @submit="onSubmit" @reset="onReset" v-if="status == 'idle'">

			       <b-form-input id="inputEventID"
                      type="hidden"
                      v-model="form.id"
                      placeholder=""  style="display: none !important;">
				   </b-form-input>

			      <b-form-group id="inputEventTitle"
			                    label="Titel"
			                    label-for="inputEventTitle"
			                    description="">

			       <b-form-input id="inputEventTitle"
                      type="text"
                      v-model="form.title"
                      required
                      placeholder="">
				   </b-form-input>
			    </b-form-group>


				<b-row>
					<b-col md="6" sm="12">
					<b-form-group id="inputEventStartDate"
				                    label="Start-Datum"
				                    label-for="inputEventStartDate"
				                    description="">

						<date-picker ref="startDatePicker" id="inputEventStartDate" v-model="form.startDate.date" :config="form.startDate.options" autocomplete="off"></date-picker>

					 </b-form-group>
				     </b-col>
				     <b-col md="6" sm="12">
					 <b-form-group id="inputEventEndDate"
				                    label="End-Datum"
				                    label-for="inputEventEndDate"
				                    description="">

						<date-picker ref="endDatePicker" id="inputEventEndDate" v-model="form.endDate.date" :config="form.endDate.options" autocomplete="off"></date-picker>

					 </b-form-group>
				     </b-col>
				 </b-row>

				<b-row>
					<b-col md="6" sm="12">

					<b-form-group id="inputEventRoomLabel"
				                    label="Raum / Ort"
				                    label-for="inputEventRoom"
				                    description="">

				      <b-form-input id="inputEventRoom"
	                      type="text"
	                      v-model="form.room"
	                      required>
					  </b-form-input>

					 </b-form-group>

				     </b-col>
				     <b-col md="6" sm="12">
					 	<b-form-group id="inputEventLocationLabel"
				                    label="Standort"
				                    label-for="inputEventLocation"
				                    description="">

					        <b-form-select id="inputEventLocation"
					                      :options="locations"
					                      required
					                      v-model="form.location">
					        </b-form-select>
						</b-form-group>
				     </b-col>
				 </b-row>

				 <b-row>
					<b-col md="6" sm="12">

					<b-form-group id="inputEventContactLabel"
				                    label="Ansprechpartner"
				                    label-for="inputEventContact"
				                    description="">

				      <b-form-input id="inputEventContact"
	                      type="text"
	                      v-model="form.organiser"
	                      required>
					  </b-form-input>

					 </b-form-group>

				     </b-col>
				     <b-col md="6" sm="12">
					 	<b-form-group id="inputEventBrandLabel"
				                    label="Veranstalter"
				                    label-for="inputEventBrand"
				                    description="">

					        <b-form-select id="inputEventBrand"
					                      :options="brands"
					                      required
					                      v-model="form.brand">
					        </b-form-select>
						</b-form-group>
				     </b-col>
				 </b-row>

			      <b-form-group id="inputEventDescription"
			                    label="Kurzbeschreibung"
			                    label-for="inputEventDescription"
			                    description="">

			       <b-form-textarea id="inputEventDescription"
                      v-model="form.description"
                      placeholder="(optional)"  style="min-height: 140px;">
				   </b-form-textarea>

			    </b-form-group>

				<b-button ref="deleteButton" id="deleteButton" size="sm" variant="link" v-if="this.form.id > 0" @click="confirmDelete" style="margin-top: 15px; font-size: 11px;"><i class="fa fa-trash"></i>&nbsp; Event löschen</b-button>


			    </b-form>

			  </b-modal>
			  </div>

	  		</div>
	  		<div v-if="this.events.length == 0 && this.status == 'idle' ">

	  		<b-list-group>
			  <b-list-group-item href="#" class="flex-column align-items-start">
			    <small>Zur Zeit gibt es keine bevorstehenden Events. <a href="#" class="btn sm" @click="openAddNew">Event erstellen</a> </small>
			  </b-list-group-item>
			</b-list-group>

	  		</div>
	  		<div v-if="this.events.length == 0 && this.status == 'init' ">

	  		<b-list-group>
			  <b-list-group-item href="#" class="flex-column align-items-start">
			    <small>Lade Events...</small>
			  </b-list-group-item>
			</b-list-group>

	  		</div>

      </b-container>

</template>

<style>
.loader {
  display: inline-block;
  border: 4px solid rgba(255,255,255,0.4);
  border-radius: 50%;
  border-top: 4px solid #17a2b8;
  width: 26px;
  height: 26px;
  -webkit-animation: spin 2s linear infinite; /* Safari */
  animation: spin 1.2s linear infinite;
}
@keyframes spin {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
}
</style>

<script>

import moment from 'moment'
import ('moment/locale/de')
import axios from 'axios'
import Config from './config/config'

import 'bootstrap/dist/css/bootstrap.css'
import 'bootstrap-vue/dist/bootstrap-vue.css'
import 'font-awesome/css/font-awesome.min.css'
import 'bootstrap4-datetimepicker/build/css/bootstrap-datetimepicker.css';
import './css/custom.css'

export default {
  name: 'app',
  props: {
	event: {
		title: String,
		startDate: String,
		description: String,
	}
  },
  data() {
	return {
		status: 'init',
		responsetxt: '',
		events: [],
		form: {
			id: '',
	        title: '',
	        startDate: {
		      date: new Date(),
			  options: {
				  format: 'DD.MM.Y HH:mm',
				  useCurrent: false,
				  sideBySide: true,
				  showClear: true,
				  showClose: true,
				  locale: 'de',
			  }
        	},
        	endDate: {
		      date: new Date(),
			  options: {
				  format: 'DD.MM.Y HH:mm',
				  useCurrent: false,
				  sideBySide: true,
				  showClear: true,
				  showClose: true,
				  locale: 'de',
			  }
        	},
	        description: '',
	        room: '',
	        location: '',
			organiser: '',
	        brand: ''
		},
		locations: [
			{ text: 'Standort wählen', value: null },
			'Berlin', 'Lübeck', 'Hamburg', 'Leipzig', 'Inning am Ammersee', 'Neßlbach/Winzer', 'Primstal', 'Weingarten', 'Regensburg', 'Bayreuth', 'München', 'Essen', 'Frankfurt am Main', 'Amsterdam'
		],
		brands: [
			{ text: 'Unternehmen wählen', value: null },
			'Hypoport', 'EUROPACE', 'Genopace', 'FINMAS', 'Dr. Klein PK', 'Dr. KLEIN FK', 'Qualitypool', 'Smart InsurTech', 'Value AG', 'Starpool', 'Vergleich.de', 'FIO'
		]
     };
  },
  filters: {
	regularDate(date) {
		return moment(date).format("DD.MM.Y");
	},
	dateDay(date) {
		return moment(date).format("DD");
	},
	dateMonth(date) {
		return moment(date).format("MM");
	},
	dateMonthName(date) {
		return moment(date).format("MMM");
	},
	dateYear(date) {
		return moment(date).format("Y");
	},
	dateTime(date) {
		return moment(date).format("HH:mm");
	},
	readableTime(date) {
		return moment.duration(date).locale('de').humanize();
	},
  },
  mounted() {
	this.getPosts();
  },
  methods: {
	getPosts() {
	      axios.get(Config.ROOT_API+'/list').then((response) => {
		    this.status = 'idle';
	        this.events = response.data;
	      }).catch( error => { console.log(error); });
	},
    onSubmit (evt) {
	  this.status = 'load';
      evt.preventDefault();

      console.log(this);

      if(!this.form.id){
	      axios.post(Config.ROOT_API+'/add', {
		      	title: this.form.title,
		        startDate: this.form.startDate.date,
	        	endDate: this.form.endDate.date,
		        description: this.form.description,
		        room: this.form.room,
		        location: this.form.location,
				organiser: this.form.organiser,
		        brand: this.form.brand
			    }
			)
		    .then(response => {
			    this.status = 'idle';
				this.$refs.modal1.hide();
			    this.getPosts();
			})
		    .catch(e => {
			   	this.status = 'idle';
			   	console.log(e);
			   	this.responsetxt = e.status;
		   });
	   } else {
	      axios.post(Config.ROOT_API+'/edit', {
		        id: this.form.id,
		      	title: this.form.title,
		        startDate: this.form.startDate.date,
	        	endDate: this.form.endDate.date,
		        description: this.form.description,
		        room: this.form.room,
		        location: this.form.location,
				organiser: this.form.organiser,
		        brand: this.form.brand
			    }
			)
		    .then(response => {
			    this.status = 'idle';
				this.$refs.modal1.hide();
				this.getPosts();
		    })
		    .catch(e => {
			   	this.status = 'idle';
				console.log(e);
		   });
	   }

    },
    onReset (evt) {
      evt.preventDefault();
      /* Reset our form values */
      this.form.title = '';
      this.form.id = null;

      	this.$refs.startDatePicker.$el.value = moment(new Date()).format('DD.MM.Y HH:00');
	    this.form.startDate.date =  new Date();
	    this.$refs.endDatePicker.$el.value = moment(new Date()).format('DD.MM.Y HH:00');
	    this.form.endDate.date =  new Date();

	    this.form.description = '';
	    this.form.room = '';
	    this.form.location = '';
		this.form.organiser = '';
	    this.form.brand = '';

    },
    assignEdit(event){

		this.form.id = event.id;
	    this.form.title = event.title;

	    //this.form.startDate.set(event.startDate);
	    this.$refs.startDatePicker.$el.value = moment(event.startDate).format('DD.MM.Y HH:mm');
	    this.form.startDate.date =  new Date(event.startDate);
	    this.$refs.endDatePicker.$el.value = moment(event.endDate).format('DD.MM.Y HH:mm');
	    this.form.endDate.date =  new Date(event.endDate);

	    this.form.description = event.description;
	    this.form.room = event.room;
	    this.form.location = event.location;
		this.form.organiser = event.organiser;
	    this.form.brand = event.brand;

	    this.$refs.modal1.show()
    },
    modalTitle() {
	    if(this.form.id){
		    return "Event bearbeiten";
	    } else {
		    return "Event hinzufügen";
	    }
    },
    openAddNew(ev){
	  this.onReset(ev);
	  this.$refs.modal1.show()
    },
    confirmDelete(){
	    var r = confirm("Dieses Event wirklich löschen?");
		if (r == true) {
			axios.post(Config.ROOT_API+'/delete', {
		        id: this.form.id
			    }
			)
		    .then(response => {
			    this.status = 'idle';
				this.$refs.modal1.hide();
				this.getPosts();
		    })
		    .catch(e => {
			   	this.status = 'idle';
				console.log(e);
		   });
		}
    }

   }
}
</script>
