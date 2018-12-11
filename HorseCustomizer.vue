<template lang="pug">
div(class="horses-wrap purple-bg horse-generator-container")
  div(class="horse-generator-header")
    img(class="logo", src="/img/layout-graphics/lab.png")
    h1 Laboratory
    p Horse Creator
    div(class="horse-name-wrap")
      h2(class="choice-title")
        span(v-if="selected.name.length > 0") {{ selected.name }} - 
        span() {{ selected.breed.title }}
        span(v-if="selected.gender")  - {{ selected.gender.name }} 
  div(class="row")
    div(id="step1")
      div(class="col-xs-12 col-lg-3 left-ui-toolbar")
        div(v-show="activeStep == 1")
          horse-breed-select(:breeds="horse_breeds", :selected="selected", v-on:breed-updated="breedUpdated($event)")
          div(class="clearfix")
          horse-gender(:genders="horse_genders", :selected="selected", v-on:gender-updated="genderUpdated($event)")
        div(v-show="activeStep == 2")
          facial-markings(:markings="active_facial_markings", :selected="selected", v-on:facial-marking-updated="facialMarkingUpdated($event)", v-on:facial-marking-removed="facialMarkingRemoved($event)")
          div(class="clearfix")
        div(v-show="activeStep == 3")
          front-left-leg-markings(:markings="active_front_left_leg_markings", :selected="selected", v-on:front-left-leg-marking-updated="frontLeftLegMarkingUpdated($event)", v-on:front-left-leg-marking-removed="frontLeftLegMarkingRemoved($event)")
          front-right-leg-markings(:markings="active_front_right_leg_markings", :selected="selected", v-on:front-right-leg-marking-updated="frontRightLegMarkingUpdated($event)", v-on:front-right-leg-marking-removed="frontRightLegMarkingRemoved($event)")
          div(class="clearfix")
      div(class="col-xs-12 col-lg-6 horse-img-container")
        img(:src="horse_image_url" v-on:load="loaderCallback" v-bind:class="{ 'img-loading': loading == true }")
        div(v-if="loading" v-bind:class="{ 'loading': loading == true }") Loading
      div(class="col-xs-12 col-lg-3 right-ui-toolbar")
        div(v-show="activeStep == 1")
          div(class="clearfix")
          horse-eye-color-select(:eye_colors="active_eye_colors", :selected="selected", v-on:eye-color-updated="eyeUpdated($event)")
          div(class="clearfix")
          horse-coat-select(:coats="active_coats", :selected="selected", v-on:coat-updated="coatUpdated($event)")
          div(class="clearfix")
        div(v-show="activeStep == 2")
          body-markings(:markings="active_body_markings", :selected="selected", v-on:body-marking-updated="bodyMarkingUpdated($event)", v-on:body-marking-removed="bodyMarkingRemoved($event)")
          div(class="clearfix")
        div(v-show="activeStep == 3")
          back-left-leg-markings(:markings="active_back_left_leg_markings", :selected="selected", v-on:back-left-leg-marking-updated="backLeftLegMarkingUpdated($event)", v-on:back-left-leg-marking-removed="backLeftLegMarkingRemoved($event)")
          back-right-leg-markings(:markings="active_back_right_leg_markings", :selected="selected", v-on:back-right-leg-marking-updated="backRightLegMarkingUpdated($event)", v-on:back-right-leg-marking-removed="backRightLegMarkingRemoved($event)")
          div(class="clearfix")
        h3(class="choice-title genetic-string") Genetic String:
        input(type="text" disabled v-model="geneString")
      div(class="clearfix")
      input(type="text", id="horse_name", v-model="selected.name", autofocus, placeholder="Choose a name", ref="horse_name")
      button(type="submit" id="submit_generate_horse" v-on:click="submit()")
        div(class="inner-lab-pass-wrap")
          img(src="/img/items/special/lab%20pass.png")
        span() Redeem
    page-nav(:active-step="activeStep", :coats="horse_coats", v-on:step-change="stepChange($event)")   
</template>

<script>
    import HorseBreedSelect from './components/HorseBreedSelect.vue'
    import HorseCoatSelect from './components/HorseCoatSelect.vue'
    import HorseGender from './components/HorseGender.vue'
    import HorseEyeColorSelect from './components/HorseEyeColorSelect.vue'
    import FacialMarkings from './components/FacialMarkings.vue'
    import BodyMarkings from './components/BodyMarkings.vue'
    import FrontLeftLegMarkings from './components/FrontLeftLegMarkings.vue'
    import FrontRightLegMarkings from './components/FrontRightLegMarkings.vue'
    import BackLeftLegMarkings from './components/BackLeftLegMarkings.vue'
    import BackRightLegMarkings from './components/BackRightLegMarkings.vue'
    import PageNav from './components/PageNav.vue'

    export default {
      data () {
        return {
          horse_breeds: JSON.parse(document.getElementById('horse_breeds_json').textContent),
          audio: false,
          loading: false,
          selected: {
            breed: null,
            name: '',
            gender: null,
            coat: null,
            eye_color: null,
            facial_markings: [],
            body_markings: [],
            front_left_leg_marking: null,
            front_right_leg_marking: null,
            back_left_leg_marking: null,
            back_right_leg_marking: null
          },
          activeStep: 1,
          horse_genders: [
            {
              id: 1,
              name: 'Stallion'
            },
            {
              id: 2,
              name: 'Mare'
            }
          ],
          eye_colors: [
            {
              name: 'Amber',
              id: 1,
              bg_color: ''
            },
            {
              name: 'Blue',
              id: 2,
              bg_color: ''
            },
            {
              name: 'Brown',
              id: 3,
              bg_color: ''
            },
            {
              name: 'Green',
              id: 4,
              bg_color: ''
            },
            {
              name: 'Hazel',
              id: 5,
              bg_color: ''
            }
          ],
          horse_coats: [],
          horse_markings: [],
          active_coats: null,
          active_facial_markings: [],
          active_body_markings: [],
          active_front_left_leg_markings: [],
          active_front_right_leg_markings: [],
          active_back_left_leg_markings: [],
          active_back_right_leg_markings: [],
          active_eye_colors: null,
          geneString: null,
          horse_image_url: null
        }
      },
      components: {
        HorseBreedSelect,
        HorseCoatSelect,
        HorseGender,
        HorseEyeColorSelect,
        FacialMarkings,
        BodyMarkings,
        FrontLeftLegMarkings,
        FrontRightLegMarkings,
        BackLeftLegMarkings,
        BackRightLegMarkings,
        PageNav
      },
      methods: {
        breedUpdated(breed) {
          this.selected.breed = breed
          //set the correct horse coats to display
          this.setActiveCoats();
          this.setActiveMarkings();
          this.update();
        },
        coatUpdated(coat) {
          this.selected.coat = coat
          this.update();
        },
        eyeUpdated(eye) {
          this.selected.eye_color = eye
          this.updateGenetics()
          this.getHorseImageUrl()
        },
        genderUpdated(gender) {
          this.selected.gender = gender
        },
        stepChange (step){
          this.activeStep = step;
        },
        facialMarkingUpdated (marking){
          this.selected.facial_markings.push(marking)
          this.update()
        },
        facialMarkingRemoved (marking){
          this.selected.facial_markings = this.selected.facial_markings.filter(function(val){
            if(val != marking){
              return true
            } else {
              return false
            }
          })
          this.update()
        },
        bodyMarkingUpdated (marking){
          this.selected.body_markings.push(marking)
          this.update()
        },
        bodyMarkingRemoved (marking){
          this.selected.body_markings = this.selected.body_markings.filter(function(val){
            if(val != marking){
              return true
            } else {
              return false
            }
          })
          this.update()
        },
        frontLeftLegMarkingUpdated (marking){
          this.selected.front_left_leg_marking = marking
          this.update()
        },
        frontLeftLegMarkingRemoved (marking){
          this.selected.front_left_leg_marking = null
          this.update()
        },
        frontRightLegMarkingUpdated (marking){
          this.selected.front_right_leg_marking = marking
          this.update()
        },
        frontRightLegMarkingRemoved (marking){
          this.selected.front_right_leg_marking = null
          this.update()
        },
        backLeftLegMarkingUpdated (marking){
          this.selected.back_left_leg_marking = marking
          this.update()
        },
        backLeftLegMarkingRemoved (marking){
          this.selected.back_left_leg_marking = null
          this.update()
        },
        backRightLegMarkingUpdated (marking){
          this.selected.back_right_leg_marking = marking
          this.update()
        },
        backRightLegMarkingRemoved (marking){
          this.selected.back_right_leg_marking = null
          this.update()
        },
        setupImgLoader: function(){
          this.loading = true;
        },
        loaderCallback: function() {
          this.loading = false;
        },
        setActiveCoats: function() {
          this.active_coats = this.horse_coats[this.selected.breed.id];  
          this.selected.coat = this.active_coats[0];
        },
        setActiveMarkings: function() {
          var that = this
          var front_left_leg_markings = []
          var front_right_leg_markings = []
          var back_left_leg_markings = []
          var back_right_leg_markings = []
          var facial_markings = []
          var body_markings = []
          for(var i=0; i < this.horse_markings[this.selected.breed.id].length;i++) {
            if(this.horse_markings[this.selected.breed.id][i].marking_type == 'leg') {
              if(this.horse_markings[this.selected.breed.id][i].marking_leg == 'FL') {
                front_left_leg_markings.push(this.horse_markings[this.selected.breed.id][i])
              } else if(this.horse_markings[this.selected.breed.id][i].marking_leg == 'FR') {
                front_right_leg_markings.push(this.horse_markings[this.selected.breed.id][i])
              } else if(this.horse_markings[this.selected.breed.id][i].marking_leg == 'BL') {
                back_left_leg_markings.push(this.horse_markings[this.selected.breed.id][i])
              } else if(this.horse_markings[this.selected.breed.id][i].marking_leg == 'BR') {
                back_right_leg_markings.push(this.horse_markings[this.selected.breed.id][i])
              }
            } else if(this.horse_markings[this.selected.breed.id][i].marking_type == 'body') {
              body_markings.push(this.horse_markings[this.selected.breed.id][i])
            } else if(this.horse_markings[this.selected.breed.id][i].marking_type == 'facial') {
              facial_markings.push(this.horse_markings[this.selected.breed.id][i])
            }
          }


          that.active_front_left_leg_markings = front_left_leg_markings
          that.active_front_right_leg_markings = front_right_leg_markings
          that.active_back_right_leg_markings = back_right_leg_markings
          that.active_back_left_leg_markings = back_left_leg_markings
          that.active_facial_markings = facial_markings;  
          that.active_body_markings = body_markings;
          that.selected.facial_markings = [];  
          that.selected.facial_markings.push(this.active_facial_markings[0]);
          that.selected.body_markings = [];
          that.selected.body_markings.push(this.active_body_markings[0]);
          that.selected.front_left_leg_marking = null;
          that.selected.front_right_leg_marking = null;
          that.selected.back_right_leg_marking = null;
          that.selected.back_left_leg_marking = null;
        },
        setActiveEyeColors: function() {
          var that = this;
          // this.active_coats = this.horse_coats[this.selected.breed.id];
          var currentlySelected = this.selected.eye_color;
            that.active_eye_colors = that.eye_colors.filter(function(color){

              if(!that.checkGeneSection('dun')){
                if(color.name == 'Amber'){
                  return false;
                }
              }

              if(!that.checkGeneSection('champagne')){
                if(color.name == 'Green'){
                  return false;
                }
              }

              if(that.selected.breed.title == 'Arabian'){
                if(color.name == 'Amber'){
                  return false;
                }
                if(color.name == 'Green'){
                  return false;
                }
              } else if(that.selected.breed.title == 'Thoroughbred'){
                if(color.name == 'Green'){
                  return false;
                }
              }

                if(that.geneString){
                  var geneParts = that.geneString.split('/')
                  var cream_dom = false;

                  for(var i = 0;i < geneParts.length; i++){
                    var part = geneParts[i]
                    if (part == 'CrCr') {
                      cream_dom = true;
                    } 
                  }

                  if(cream_dom && color.name != 'Blue'){
                    return false;
                  } 
                }
                

              return true;
            });

            for (var i = 0; i < that.active_eye_colors.length; i++) {
              if (currentlySelected == that.active_eye_colors[i]) {
                that.selected.eye_color = that.active_eye_colors[i];
                return;
              }
            };
            that.selected.eye_color = that.active_eye_colors[0];
          // var activeColors = this.eye_colors.splice(0);
          // if(this.selected.breed.title == 'Arabian'){
          //   activeColors.forEach(function(color, index, array){
          //     if(color.name == 'Amber'){
          //       activeColors = array.splice(index, 1);
          //     }
          //     if(color.name == 'Green'){
          //       activeColors = array.splice(index, 1);
          //     }
          //   });
          // }
          // this.active_eye_colors = activeColors;
        },
        getHorseCoats: function(id) {
          if(id){
            switch(id) {
                case 7:
                    this.horse_coats[id] = JSON.parse(document.getElementById('andalusian_coats_json').textContent)
                    this.horse_markings[id] = JSON.parse(document.getElementById('andalusian_markings_json').textContent)
                    break
                case 2:
                    this.horse_coats[id] = JSON.parse(document.getElementById('warmblood_coats_json').textContent)
                    this.horse_markings[id] = JSON.parse(document.getElementById('warmblood_markings_json').textContent)
                    break
                case 3:
                    this.horse_coats[id] = JSON.parse(document.getElementById('arabian_coats_json').textContent)
                    this.horse_markings[id] = JSON.parse(document.getElementById('arabian_markings_json').textContent)
                    break
                case 4:
                    this.horse_coats[id] = JSON.parse(document.getElementById('thoroughbred_coats_json').textContent)
                    this.horse_markings[id] = JSON.parse(document.getElementById('thoroughbred_markings_json').textContent)
                    break
                case 6:
                    this.horse_coats[id] = JSON.parse(document.getElementById('americanquarter_coats_json').textContent)
                    this.horse_markings[id] = JSON.parse(document.getElementById('americanquarter_markings_json').textContent)
                    break        
            }
          }  
        },
        getHorseImageUrl: function(){
          var url ='?',
              that = this

          this.loading = true;

          url += 'breed_id='+this.selected.breed.id;
          url += '&coat_id='+this.selected.coat.id;
          url += '&gender='+this.selected.gender.name;
          url += '&eye_color='+this.selected.eye_color.name;
          if(that.selected.facial_marking)
            url += '&markings[]='+ that.selected.facial_marking.id;
          //set markings
          for(var i=0; i < that.selected.body_markings.length; i++){
            url += '&markings[]='+ that.selected.body_markings[i].id;
          }

          for(var i=0; i < that.selected.facial_markings.length; i++){
            url += '&markings[]='+ that.selected.facial_markings[i].id;
          }

          if(that.selected.front_left_leg_marking)
            url += '&markings[]='+ that.selected.front_left_leg_marking.id;

          if(that.selected.front_right_leg_marking)
            url += '&markings[]='+ that.selected.front_right_leg_marking.id;

          if(that.selected.back_left_leg_marking)
            url += '&markings[]='+ that.selected.back_left_leg_marking.id;

          if(that.selected.back_right_leg_marking)
            url += '&markings[]='+ that.selected.back_right_leg_marking.id;

          // for(var i=0; i < that.selected.leg_markings.length; i++){
          //   url += '&markings[]='+ that.selected.leg_markings[i].id;
          // }

          this.horse_image_url = '/Horses/ajaxDisplayImage' + url;
        },
        parseHorseCoats: function(){
          this.getHorseCoats(7);
          this.getHorseCoats(2);
          this.getHorseCoats(3);
          this.getHorseCoats(4);
          this.getHorseCoats(6);
        },
        update () {
          this.updateGenetics();
          this.setActiveEyeColors();
          this.getHorseImageUrl()
        },
        toggleAudio () {
          var player = document.getElementById("music");
          this.audio = !this.audio;
          if(this.audio){
            player.play();
          } else {
            player.pause();
          }
        },
        updateGenetics: function(){
          var base = this.geneticBase();
          if(!base){
            this.geneString = null;
          }
          this.geneString = base;

        },
        replaceGeneStringSection: function(target, newVal, geneString){
          if(!geneString || geneString.length < 2)
            return;

          var geneParts = geneString.split('/');

          if(target == 'rbf'){
            if(geneParts && geneParts.length > 0){
              geneParts[0] = newVal;  
            } else {
              return newVal;
            }
          }

          if(target == 'agouti'){
            if(geneParts && geneParts.length > 1){
              geneParts[1] = newVal;  
            } else {
              if(geneString)
                return geneString + '/' + newVal;
              else
                return 'EE/' + newVal;
            }
          }

          return geneParts.join('/');

        },
        geneticBase: function(){
          var that = this;
          if(!this.selected.coat){
            return null;
          }

          var baseCoat = this.selected.coat.title.trim();
          var baseCoatParts = baseCoat.split(' '); 


          var g_string = '';

          var coatPartsActive = [];
          baseCoatParts.forEach(function(element, index){
            coatPartsActive[element.toLowerCase()] = true;
          });

          //determine base color / redblack factor
          if(coatPartsActive['black'] && !coatPartsActive['bay']){
            g_string = 'EE/aa';
          } else if(coatPartsActive['bay']){
            g_string = 'EE/AA';
          } else if(coatPartsActive['chestnut']){
            g_string = 'ee/AA';
          } else if(coatPartsActive['amber']){
            g_string = 'EE/AA';
          } else {
            g_string = 'Ee/Aa';
          }

          //check for Dun related genes
          if(coatPartsActive['dun']){
            g_string += '/DD';
          } else if(coatPartsActive['grullo']){
            g_string += '/DD';
          } else if(coatPartsActive['red'] && coatPartsActive['dun']){
            g_string += '/DD';
          }

          //check for dunskin
          if(coatPartsActive['dunskin']){
            g_string = that.replaceGeneStringSection('rbf', 'EE', g_string);
            g_string = that.replaceGeneStringSection('agouti', 'AA', g_string);
            g_string += '/Dd/ncR';
          }

          //check for cream gene and add
          if(coatPartsActive['cream']){
            g_string = that.replaceGeneStringSection('rbf', 'EE', g_string);
            g_string = that.replaceGeneStringSection('agouti', 'aa', g_string);
            g_string += '/CrCr';
          } else if(coatPartsActive['palomino']){
            g_string = that.replaceGeneStringSection('rbf', 'ee', g_string);
            g_string += '/nCr';
          } else if(coatPartsActive['buckskin']){
            g_string = that.replaceGeneStringSection('rbf', 'EE', g_string);
            g_string = that.replaceGeneStringSection('agouti', 'Aa', g_string);
            g_string += '/nCr';
          } else if(coatPartsActive['smoky']){
            g_string = that.replaceGeneStringSection('rbf', 'Ee', g_string);
            g_string = that.replaceGeneStringSection('agouti', 'aa', g_string);
            g_string += '/nCr';
          } else if(coatPartsActive['cremello']){
            g_string = that.replaceGeneStringSection('rbf', 'ee', g_string);
            g_string = that.replaceGeneStringSection('agouti', 'Aa', g_string);
            g_string += '/CrCr';
          } else if(coatPartsActive['perlino']){
            g_string = that.replaceGeneStringSection('rbf', 'EE', g_string);
            g_string = that.replaceGeneStringSection('agouti', 'Aa', g_string);
            g_string += '/CrCr';
          }

          //check for dunalino
          if(coatPartsActive['dunalino']){
            g_string = that.replaceGeneStringSection('rbf', 'ee', g_string);
            g_string = that.replaceGeneStringSection('agouti', 'Aa', g_string);
            g_string += '/DD/ncR';
          }

          //check champagne genes
          if(coatPartsActive['champagne']){
            //determine champagne type
            if(coatPartsActive['golden']){
              g_string = that.replaceGeneStringSection('rbf', 'ee', g_string);
            } else if(coatPartsActive['classic']){
              g_string = that.replaceGeneStringSection('rbf', 'Ee', g_string);
              g_string = that.replaceGeneStringSection('agouti', 'aa', g_string);
            } else if(coatPartsActive['amber']){
              g_string = that.replaceGeneStringSection('rbf', 'Ee', g_string);
              g_string = that.replaceGeneStringSection('agouti', 'AA', g_string);
            }

            g_string += '/ChCh';
          }

          //check white genes
          if(coatPartsActive['white']){
            g_string += '/Ww';
          }

          //check grey
          if(coatPartsActive['grey']){
            g_string += '/Gg';
          }

          //check pearl
          if(coatPartsActive['pearl']){
            if(coatPartsActive['cream']){
              g_string += '/nPrl';
            } else {
              g_string += '/PrlPrl';
            }
          }

          //check silver
          if(coatPartsActive['silver']){
            g_string = that.replaceGeneStringSection('rbf', 'Ee', g_string);
            g_string += '/nZ';
          }

          //check roan
          if(coatPartsActive['roan']){
            g_string += '/Rr';
          }

          //check appaloosa
          if(coatPartsActive['Appaloosa']){
            g_string += '/nLp';
          }

          return g_string;
        },
        checkGeneSection(section){
          var that = this;

          if(!that.geneString || that.geneString == null){
            return false;
          }

          var geneParts = that.geneString.split('/')
          for(var i = 0;i < geneParts.length; i++){
            var part = geneParts[i]
            if ( section == 'dun') {
              if (part.toLowerCase() == 'dd') {
                return true;
              }
            } else if ( section == 'cream') {
              if (part.toLowerCase() == 'ncr' || part.toLowerCase() == 'crcr') {
                return true;
              } 
            } else if ( section == 'champagne') {
              if (part.toLowerCase() == 'chch' || part.toLowerCase() == 'chch' ) {
                return true;
              } 
            }  
          }

          return false;

        },
        submit () {
          var that = this;

          var valid = that.validate();

          if(!valid.status){
            setTimeout(function(){
                swal({
                  title: "Error",
                  text: valid.msg,
                  type: "error"
                });
                return;   
            }, 0);
            return;
          }

          swal({
            title: "Finalize Horse",
            text: "Submit to finalize horse and save to your account. 1 Lab Pass will be deducted from your inventory.",
            type: "info",
            showCloseButton: true,
            showLoaderOnConfirm: true,
            preConfirm: () => {
              var reqData = JSON.stringify(that.selected);
              return $.ajax({
                url: "/Horses/submitHorseGenerator",
                method: "POST",
                data: reqData,
                dataType: "json"
              })

              .done(function( msg ) {
                if(msg && msg.success){
                  setTimeout(function(){
                    swal({
                      title: 'Horse Created!',
                      text: msg.success.msg,
                      type: 'success'
                    });   
                  }, 0);
                }
              })

              .fail(function( jqXHR, textStatus ) {
                var resp = JSON.parse(jqXHR.responseText)
                setTimeout(function(){
                    swal({
                      title: 'Unable to generate Horse',
                      text: resp.error.msg,
                      type: 'error'
                    });   
                  }, 0);
              })

            },
            allowOutsideClick: true 
          }, function(){
            var reqData = JSON.stringify(that.selected);
            var request = $.ajax({
              url: "/Horses/submitHorseGenerator",
              method: "POST",
              data: reqData,
              dataType: "json"
            });
             
            request.done(function( msg ) {
              if(msg && msg.success){
                setTimeout(function(){
                  swal({
                    title: 'Horse Created!',
                    text: msg.success.msg,
                    type: 'success'
                  });   
                }, 0);
              }
            });
             
            request.fail(function( jqXHR, textStatus ) {
              var resp = JSON.parse(jqXHR.responseText)
              setTimeout(function(){
                  swal({
                    title: 'Unable to generate Horse',
                    text: resp.error.msg,
                    type: 'error'
                  });   
                }, 0);
            });
          })
        },
        validate () {
          var that = this,
              returnObj = {
                status: false,
                msg: ''
              }

          if(!this.selected.name || this.selected.name.length < 1){
            returnObj.status = false;
            returnObj.msg = 'You must provide a name for your horse.';
            return returnObj;
          }

          if(!this.selected.breed || !this.selected.eye_color || !this.selected.coat){
            returnObj.status = false;
            returnObj.msg = 'You must choose a horse breed, coat, gender, and eye color.';
            return returnObj;
          }

          returnObj.status = true;
          returnObj.msg = 'Horse choices are valid.';
          return returnObj;
        }
      },
      created() {
        this.selected.breed = this.horse_breeds[0]
        this.parseHorseCoats()
      },
      mounted: function() {
        this.$refs.horse_name.focus()
        this.setActiveCoats()
        this.setActiveMarkings()
        this.selected.gender = this.horse_genders[0];
        this.setActiveEyeColors()
        this.selected.eye_color = this.active_eye_colors[0];
        this.update()

        //start music
        // this.toggleAudio()
      }
    }
</script>

<style lang="stylus">
font-stack = Helvetica, sans-serif
primary-color = #999
div.horse-generator-container
  padding: 5px 5px 140px;
  height auto
  width 100%
  h1,h2,h3
    margin-top: 0
  .container
    margin-top 50px
    min-height 625px
  .left-ui-toolbar
  .right-ui-toolbar
  .horse-generator-header
    text-align center
    h1
      display inline-block
      font-size 40px
      text-shadow 2px 2px 3px #381859
      vertical-align top
      margin-top 4px
      margin-bottom 4px
      position relative
    p
      font-size 20px
      letter-spacing 10px
      text-align center
      padding-left 48px
      color white
      text-shadow 2px 1px 1px #381859
    .audio-control-wrap
      position: absolute;
      right: -54px;
      top: 8px;
      font-size: 24px;
      cursor: pointer;
      color: #252020;
      text-align: left;
      width: 30px;
      &.active
        color white
        .fa-volume-off
          display none!important
        .fa-volume-up
          display block
      .fa-volume-up
        display: none

    .horse-name-wrap
      margin-top 28px
      .choice-title
        font-size 26px  
  .choice-title
    color: #fff;
    text-align: center;
    display: block;
    margin-bottom: 8px;
    text-shadow: 2px 2px 1px #381859;
    font-family: 'Source Sans Pro';
    &.gender
      margin-top: 4px;
      margin-left: 4px;
      display: inline-block;
    &.genetic-string
      margin-top 20px  
  #horse_name
    position: absolute;
    bottom: 100px;
    color #fff
    border none
    background none
    outline none
    font-size 30px
    text-align center
    margin-top 0
    font-weight 100;
    &::-webkit-input-placeholder
      color white
    &:-moz-placeholder /* Firefox 18- */
      color white
    &::-moz-placeholder/* Firefox 19+ */
      color white
    &:-ms-input-placeholder/* IE 10+ */
      color white
    &::-ms-input-placeholder /* Edge */
      color white
    &:placeholder-shown/* Standard one last! */
      color white
  .choices-wrap
    background rgba(245,220,245,0.76)
    border-radius 10px
    padding 5px
    margin-bottom 10px
    li
      height 82px
      width 42%
      background rgba(255, 255, 255, 0.47)
      display inline-block
      margin 5px 5px 5px
      border-radius 10px
      opacity 0.5
      cursor pointer
      vertical-align: top
      &:hover
        opacity 1
      &.selected
        opacity 1
        box-shadow 0px 0px 12px 4px rgb(65, 184, 131)
        box-sizing border-box
      img
        width 100%
        text-align center
        display block
        margin 0 auto
        border-bottom-right-radius 10px       
.logo
  width 40px
  height 40px
  margin-top 4px
  margin-right 5px
  vertical-align top
.horse-img-container
  img
    width 100%
    &.img-loading
      opacity 0
.loading {
  color: white;
  text-align: center;
  font-size: 30px;
}

.loading:after {
  overflow: hidden;
  display: inline-block;
  vertical-align: bottom;
  -webkit-animation: ellipsis steps(4,end) 900ms infinite;      
  animation: ellipsis steps(4,end) 900ms infinite;
  content: "\2026"; /* ascii code for the ellipsis character */
  width: 0px;
}

#submit_generate_horse
  outline: none;
  border: 0;
  width: 300px;
  text-align: center;
  background: #a3ceba;
  border-radius: 14px;
  position: absolute;
  bottom: 10px;
  right: 10px;
  padding: 5px;
  box-shadow: 1px 1px 1px rgba(67, 63, 63, 0.74);
  cursor pointer
  .inner-lab-pass-wrap
    display: inline-block;
    height: auto;
    background: #e3435f;
    width: 55px;
    text-align: center;
    border-radius: 20px;
    box-shadow: 0px 0px 7px 2px #e3435f;
    margin: 5px;
    img
      width 80%
      display inline-block
      margin-top 6px
      text-align center
  span
    color #e3435f
    font-weight 600
    font-size 3em
    letter-spacing 2px
    text-shadow 1px 1px 1px rgba(0, 0, 0, 0.74)
    line-height 59px
    display: inline-block
    vertical-align top
    margin-left 10px

@keyframes ellipsis {
  to {
    width: 1.25em;    
  }
}

@-webkit-keyframes ellipsis {
  to {
    width: 1.25em;    
  }
}
</style>