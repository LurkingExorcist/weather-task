<template>
  <div id="app" 
    v-touch:swipe="swipe">
    <div class="loading" v-if="days.length == 0">
      <div class="error" v-if="errorText">
        <div class="error-header">Ошибка!</div>
        <div class="error-text">{{errorText}}</div>
      </div>
      <div class="loading-text" v-else>
        Загрузка...
      </div>
    </div>
    <div class="container" v-else-if="days.length > 0" >
      <div class="date">
        <transition 
          :enter-class="prevActive < active ? 'slide-enter' : 'slide-leave-to'"
          :leave-to-class="prevActive < active ? 'slide-leave-to' : 'slide-enter'"
          name="date-slide"
        >
          <span :key="active" class="date-text">
            {{ states[active] }}, {{ days[active].dateText }} {{ days[active].year }}
          </span>
        </transition>
      </div>
      <div class="content">
        <transition 
          :enter-class="prevActive < active ? 'slide-enter' : 'slide-leave-to'"
          :leave-to-class="prevActive < active ? 'slide-leave-to' : 'slide-enter'"
          name="description-slide"
        >
          <div class="weather-status" :key="active">
            {{ days[active].description }}
          </div>
        </transition>
        <transition 
          :enter-class="prevActive < active ? 'slide-enter' : 'slide-leave-to'"
          :leave-to-class="prevActive < active ? 'slide-leave-to' : 'slide-enter'"
          name="icon-slide"
        >
          <div class="ellipse e-1" :key="active">
            <div class="ellipse">
              <div class="ellipse">
                <div class="ellipse">
                  <div class="ellipse">
                    <div class="weather-icon" v-html="days[active].svg.default">
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </transition>
        <transition 
          :enter-class="prevActive < active ? 'slide-enter' : 'slide-leave-to'"
          :leave-to-class="prevActive < active ? 'slide-leave-to' : 'slide-enter'"
          name="temp-slide"
        >
          <div class="temp" :key="active">
            {{ days[active].temp }}<span class="degree"></span>
          </div>
        </transition>

        <div class="city">
          Казань
        </div>
        <div class="bottom-panel">
          <div 
            v-for="({ dateText, svg }, i) in days"
            :class="`day ${active == i ? 'active' : ''}`"
            :key="i"
            @click="switchDay(i)"
          >
            <div class="day-icon" v-html="svg.default"></div>
            {{ dateText }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import axios from 'axios'
  import moment from 'moment'
  import './assets/style.scss'

  moment.locale('ru');

  export default {
    name: 'app',

    data(){
      return {
        temp: '22',
        apikey: 'f3731714fa6d1827944d536526c61319',
        weatherdata: null,
        days: [],
        active: 0,
        prevActive: -1,

        states: [
          'Сегодня', 'Завтра', 'Послезавтра'
        ],

        tomorrow: moment().add(2, 'days'),

        errorText: null
      }
    },

    computed: {
      curDate(){
        return moment().format('DD MMMM YYYY')
      }
    },

    async created(){
      await this.getWeatherData();
    },

    methods: {
      swipe(e){
        this.prevActive = this.active; 

        if(e == 'left'){
          if(this.active + 1 < this.days.length)
            this.active++;
        }
        else{
          if(this.active - 1 > -1)
            this.active--;
        }
      },
      switchDay(i){
        if(i > -1 && i < this.days.length){
          this.prevActive = this.active; 
          this.active = i;
        }
      },
      ltAfterTomorrow(date){
        return date.isBefore(this.tomorrow)
      },
      capitalize(text){
        return text.split('')[0].toUpperCase() + text.slice(1);
      },
      async getWeatherData(){
        try{
          const { data: { list } } = await axios.get(`https://api.openweathermap.org/data/2.5/forecast?id=551487&apikey=${this.apikey}&cnt=24&lang=ru&units=metric`);

          this.tomorrow.set('hour', 23);

          let days = list.map((d, i) => ({
            date: moment(d.dt*1000),
            dateText: moment(d.dt*1000).format(`DD MMMM`),
            year: moment(d.dt*1000).format(`YYYY`),
            hours: moment(d.dt*1000).format(`HH`),
            temp: parseInt(d.main.temp),
            description: this.capitalize(d.weather[0].description),
            svg: require(`!raw-loader!./assets/images/${d.weather[0].icon}.svg`)
          }));

          if(+days[0].hours > 12){
            this.days.push(days[0]);
            days.splice(0, 1);
          }

          this.days = this.days
            .concat(days.filter(d => d.hours == '12' && this.ltAfterTomorrow(d.date)));
        }catch(e){
          console.log(e);
          if(e.response && e.response.data){
            this.errorText = e.response.data.message;
          }else{
            this.errorText = e.message || e;
          }
        }
      }
    }
  }
</script>
