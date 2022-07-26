/* eslint-disable */
<template>
  <div class="events">
    <h1>Events for Good</h1>
    <EventCard v-for="event in events" :key="event.id" :event="event" />

    <div class="pagination">
      <router-link
        id="page-prev"
        :to="{ name: 'EventList', query: { page: page - 1 } }"
        rel="prev"
        v-if="page != 1"
        >&#60; Previous
      </router-link>

      <router-link
        id="page-next"
        :to="{ name: 'EventList', query: { page: page + 1 } }"
        rel="next"
        v-if="hasNextPage"
      >
        Next &#62;
      </router-link>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
import EventCard from "@/components/EventCard.vue";
import EventService from "@/services/EventService.js";

export default {
  name: "EventList",
  props: ["page"],
  components: {
    EventCard,
  },
  data() {
    return {
      events: null,
      totalEvents: 0,
    };
  },
  //Lifecycle hook (component have more lifecycle hooks), when the component is created it calls the server for the events stored in the
  //mocked db on the URL below.
  //Axios (stored in EventService) is asynced and promise based so we need to wait for the request. Achieve that we use then().
  //created() {
  //watchEffect(() => {
  // This clear out the events on the page, so user knows that API has been called.
  //this.event = null;
  //EventService.getEvents(2, this.page)
  //.then((response) => {
  //this.events = response.data;
  //this.totalEvents = response.headers["x-total-count"];
  //})
  // If the request is rejected catch the situation.
  //.catch(() => {
  //this.$router.push({ name: "NetworkError" });
  //});
  //});
  //},
  // Since the component isn't loaded yet, we have no access to 'this'.
  beforeRouteEnter(routeTo, routeFrom, next) {
    // Parse the number from the route we're navigating to.
    EventService.getEvents(2, parseInt(routeTo.query.page) || 1)
      .then((response) => {
        // Continue routing and once component is loaded, set these values.
        // We are using 'comp'(as in component) in the docs is 'vm'(as in View Model).
        next((comp) => {
          comp.events = response.data;
          comp.totalEvents = response.headers["x-total-count"];
        });
      })
      .catch(() => {
        // If the API fails, load the NetworkError page.
        next({ name: "NetworkError" });
      });
  },
  beforeRouteUpdate(routeTo) {
    // Parse the number from the route we're navigating to.
    // Return the promise so Vue Router knows to wait on the API call before loeading the page.
    return EventService.getEvents(2, parseInt(routeTo.query.page) || 1)
      .then((response) => {
        //Switch to using 'this', since the component is already created.
        this.events = response.data;
        this.totalEvents = response.headers["x-total-count"];
      })
      .catch(() => {
        // If the API fails, load the NetworkError page.
        return { name: "NetworkError" };
      });
  },
  computed: {
    hasNextPage() {
      // Finds the total number of pages.
      var totalPages = Math.ceil(this.totalEvents / 2);

      // If the page is not the last page show 'Next Page' link in Pagination.
      return this.page < totalPages;
    },
  },
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
.pagination a {
  flex: 1;
  text-decoration: none;
  color: #2c3e50;
}

#page-prev {
  text-align: left;
}

#page-next {
  text-align: right;
}
</style>
