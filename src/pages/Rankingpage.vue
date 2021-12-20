<template>
  <div>
    <!-- Button -->
    <button class="cybr-btn" @click="goToPage('home')">
      Go home<span aria-hidden>_</span>
      <span aria-hidden class="cybr-btn__glitch">Go home</span>
      <span aria-hidden class="cybr-btn__tag">#1</span>
    </button>
    <!-- FlexBox div  -->
    <div class="center">
      <!-- H1  -->
      <h1>Ranking for</h1>
      <!-- Table -->
      <table>
        <tr>
          <!-- Table headers -->
          <th>Ranking for</th>
          <th>Title</th>
          <th>Artist</th>
          <th>Points</th>
          <th>Out of</th>
          <th>% of Total</th>
          <th>Spotify Refference</th>
        </tr>
        <!-- Items themselves -->
        <!-- For every song in the fetched JSON -->
        <tr v-for="(song, i) in songs" :key="i">
          <!-- Ranking number -->
          <td>#{{ i + 1 }}</td>
          <!-- Song details -->
          <td>{{ song.title }}</td>
          <td>{{ song.artist }}</td>
          <td>{{ song.totalPoints }}</td>
          <td>{{ song.allPoints }}</td>
          <td>{{ song.percentage }}%</td>
          <!-- Fetch spotify song -->
          <td>
            <iframe
              :src="song.spotify"
              height="80"
              width="100%"
              frameBorder="0"
            ></iframe>
          </td>
        </tr>
      </table>
    </div>
  </div>
</template>

<script>
export default {
  name: "Rankingpage",
  data() {
    return {
      //Initialze songs array
      songs: [],
    };
  },
  methods: {
    //Method to change page to clicked item
    goToPage(page) {
      this.$emit("change-page", page);
    },
    fetchSongs() {
      // Get all songs from Elke's API
      const url = "http://webservies.be/eurosong/Songs";

      fetch(url)
        .then((response) => {
          //JSON parse the output
          return response.json();
        })
        .then((songs) => {
          //Call both methods to get all votes and artists that are needed to be displayed
          this.fetchVotes(songs);
          this.fetchArtists(songs);
        });
    },

    fetchArtists(songs) {
      //Fetch artists from Elke's artists endpoint
      const url = "http://webservies.be/eurosong/Artists";
      fetch(url)
        .then((response) => {
          //JSON parse output
          return response.json();
        })
        .then((artists) => {
          //Map each song and get the specified artist from the nested objects
          songs.map((song) => {
            const a = artists.find((artist) => artist.id == song.artist);
            song.artist = a.name;
            return song;
          });
          this.songs = songs;
        });
    },
    fetchVotes(songs) {
      //Fetch artists from Elke's votes endpoint
      const url = "http://webservies.be/eurosong/Votes";

      fetch(url)
        .then((response) => {
          //JSON parse the output
          return response.json();
        })
        .then((votes) => {
          //Ininitalize allPoints that is going to show the total amount of points that all songs have together
          let allPoints = 0;
          //Loop over songs
          songs.map((song) => {
            //Initialize new property of class song and set it equal to 0 (this will later on get filled with correct amount of votes)
            song.totalPoints = 0;
            //Find the correct votes for the corresponding song by filtering
            votes
              .filter((vote) => vote.songID == song.id)
              //For every vote that was found add the int (of the vote) to the total points and also to the corresponding song
              .forEach((element) => {
                if (element.songID == song.id) {
                  song.totalPoints += element.points;
                  allPoints += element.points;
                }
              });
            
            //Return song with all new data added
            return song;
          });

          //Calculate for each song the % of votes it holds compared to the previously calculated total & calculate total amount of votes
          let fullTotal = 0;
          songs.forEach((song) => {
            song.percentage = Number(
              ((song.totalPoints / allPoints) * 100).toFixed(2)
            );
            fullTotal += song.totalPoints
          });

          songs.forEach((song) => {
            song.allPoints = fullTotal
          })

          //Get the correct ranking for the songs with all correct data
          this.songs = songs.sort((y, z) => z.totalPoints - y.totalPoints);
        });
    },
  },
  //On load of this page perform the mounted method
  mounted() {
    this.fetchSongs();
  },
};
</script>