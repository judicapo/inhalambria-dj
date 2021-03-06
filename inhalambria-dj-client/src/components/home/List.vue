<template>
  <div class="card">
    <div class="card-content">
      <b-table
        :data="data"
        :loading="loading"
        paginated
        backend-pagination
        :total="total"
        :per-page="perPage"
        @page-change="onPageChange"
        aria-next-label="Next page"
        aria-previous-label="Previous page"
        aria-page-label="Page"
        aria-current-label="Current page"
        backend-sorting
        :default-sort-direction="defaultSortOrder"
        :default-sort="[sortField, sortOrder]"
        @sort="onSort"
        icon-pack="fas"
      >
        <template slot-scope="props">
          <b-table-column field="track_title" label="Track title" sortable>
            {{ props.row.name }}
          </b-table-column>

          <b-table-column field="album_name" label="Album name">
            {{ props.row.album.name }}
          </b-table-column>

          <b-table-column
            field="tracks_count"
            label="Tracks count"
            numeric
            sortable
          >
            <span class="tag" :class="type(props.row.album.total_tracks)">
              {{ props.row.album.total_tracks }}
            </span>
          </b-table-column>

          <b-table-column
            field="release_date"
            label="Release Date"
            sortable
            centered
          >
            {{
              props.row.album.release_date
                ? new Date(props.row.album.release_date).toLocaleDateString()
                : ""
            }}
          </b-table-column>

          <b-table-column field="artist" label="Artist">
            {{ props.row.artists[0].name | truncate(80) }}
          </b-table-column>

          <b-table-column v-if="user()" field="" label="">
            <button
              class="button is-primary is-small"
              :disabled="props.row.enabled"
              v-on:click="
                favTrack({
                  trackId: props.row.id,
                  artist: props.row.artists[0].name,
                  release_date: props.row.album.release_date,
                  tracks_count: props.row.album.total_tracks,
                  album_name: props.row.album.name,
                  track_title: props.row.name
                })
              "
            >
              <span>fav </span>
            </button>
          </b-table-column>
        </template>
      </b-table>
    </div>
  </div>
</template>

<script>
import { mapGetters, mapActions } from "vuex";

export default {
  props: {
    mainLoadAsyncData: Function
  },
  data() {
    return {
      data: [],
      total: 0,
      loading: false,
      sortField: "tracks_count",
      sortOrder: "desc",
      defaultSortOrder: "desc",
      page: 0,
      perPage: 50
    };
  },
  methods: {
    ...mapGetters("auth", ["user"]),
    ...mapActions("favorites", {
      createFavorite: "create",
      getFavorites: "find"
    }),
    async checkTack(id) {
      try {
        const { data } = await this.getFavorites({
          query: {
            trackId: id,
            $sort: { createdAt: -1 },
            $limit: 1
          }
        });
        const res = data.length > 0;
        return res;
      } catch {
        return false;
      }
    },
    favTrack(track) {
      this.createFavorite({ ...track, userId: this.user().id })
        .then(res => {
          let selected = this.data.find(x => x.id === track.trackId);
          selected.enabled = true;
          this.$buefy.snackbar.open(`${res.track_title} added to your library`);
        })
        .catch(err => {
          if (err.message.includes("Validation error"))
            this.$buefy.snackbar.open(`You have already added it.`);
          else
            this.$buefy.snackbar.open(`there was an error, try again later!`);
        });
    },
    async loadAsyncData() {
      const params = {
        sortField: this.sortField,
        sortOrder: this.sortOrder,
        page: this.page
      };

      this.loading = true;
      const { items, total } = await this.mainLoadAsyncData(params);

      this.data = [];
      let currentTotal = total;
      if (total / this.perPage > 1000) {
        currentTotal = this.perPage * 1000;
      }
      this.total = currentTotal;
      items.forEach(async item => {
        if (!item) return;
        item.album.release_date = item.album.release_date.replace(/-/g, "/");
        item.enabled = await this.checkTack(item.id);
        this.data.push(item);
      });
      this.loading = false;
    },
    /*
     * Handle page-change event
     */
    onPageChange(page) {
      this.page = page - 1;
      this.loadAsyncData();
    },
    /*
     * Handle sort event
     */
    onSort(field, order) {
      this.sortField = field;
      this.sortOrder = order;
      this.loadAsyncData();
    },
    /*
     * Type style in relation to the value
     */
    type(value) {
      const number = parseFloat(value);
      if (number < 6) {
        return "is-danger";
      } else if (number >= 6 && number < 8) {
        return "is-warning";
      } else if (number >= 8) {
        return "is-success";
      }
    }
  },
  filters: {
    /**
     * Filter to truncate string, accepts a length parameter
     */
    truncate(value, length) {
      return value.length > length ? value.substr(0, length) + "..." : value;
    }
  },
  mounted() {
    this.loadAsyncData();
  }
};
</script>
