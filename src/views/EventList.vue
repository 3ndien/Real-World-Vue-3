<template>
  <h1>Events For Good</h1>
  <div class="events">
    <EventCard v-for="event in events" :key="event.id" :event="event" />
    <div class="pagination">
      <router-link id="page-prev" :to="{ name: 'EventList', query: { page: page - 1 } }" 
                  rel="prev"
                  v-if="page != 1">
                  &#60; Previus
    </router-link>
    <router-link id="curr" v-for="curr in allPages" :key="curr" :to="{ name: 'EventList', query: { page: curr } }"
                  rel="current">
                  {{ curr }}
    </router-link>
    <router-link id="page-next" :to="{ name: 'EventList', query: { page: page + 1 } }" 
                  rel="next"
                  v-if="hasNextPage">
                  Next &#62;
    </router-link>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
import EventCard from "@/components/EventCard.vue";
import EventService from "./../services/EventService";
import NProgress from "nprogress";

export default {
  name: "EventList",
  props: ['page'],
  components: {
    EventCard,
  },
  data() {
    return {
      events: null,
      totalEvents: 0,
    };
  },
  beforeRouteEnter(routeTo, routeFrom, next) {
    NProgress.start();
      EventService.getEvents(2, parseInt(routeTo.query.page) || 1)
      .then(response => {
        next(comp => {
          comp.totalEvents = response.headers['x-total-count']
          comp.events = response.data
        })
      })
      .catch(() => {
        next({ name: 'NetworkError' });
      })
      .finally(() => {
        NProgress.done()
      })
  },
  beforeRouteUpdate(routeTo) {
    NProgress.start();
      EventService.getEvents(2, parseInt(routeTo.query.page) || 1)
      .then(response => {
          this.totalEvents = response.headers['x-total-count']
          this.events = response.data
      })
      .catch(() => {
        return ({ name: 'NetworkError' });
      })
      .finally(() => {
        NProgress.done()
      })
  },
  computed: {
    hasNextPage() {
      var totalPages = Math.ceil(this.totalEvents / 2)

      return this.page < totalPages;
    },

    allPages() {
      return Math.ceil(this.totalEvents / 2);
    }
  }
};
</script>

<style scoped>
.events {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.pagination {
  display: flex;
  width: 290px;
}

.pagination a{
  flex: 1;
  text-decoration: none;
  color: #2c3e50;
}

#page-prev {
  text-align: left;
}

#curr {
  text-align: center;
  flex: 0.3; 
}

#page-next {
  text-align: right;
}
</style>
