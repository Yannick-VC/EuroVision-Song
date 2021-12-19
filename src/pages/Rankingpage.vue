<template>
    <div>
        <button @click="goToPage('home')">
            Go to home
        </button>
        <h1>
            Ranking
        </h1>
        <table>
            <tr>
                <th>Ranking</th>
                <th>Song Title</th>
                <th>Song Artist</th>
                <th>Total Points</th>
                <th>Total %</th>
                <th>Spotify Link</th>
            </tr>
            <tr v-for="(song, i) in songs" :key="i">
                <td>#{{i + 1}}</td>
                <td>{{song.title}}</td>
                <th>{{song.artist}}</th>
                <td>{{song.totalPoints}}</td>
                <td>{{song.percentage}}%</td>
                <td><iframe :src="song.spotify" height="80" width="100%" frameBorder="0"></iframe></td>
            </tr>
        </table>
    </div>
</template>

<script>
    export default {
        name: "Rankingpage",
        data(){
            return {
                songs: []
            }
        },
        mounted(){
            this.fetchSongs();
        },
        methods: {
            goToPage(page) {
                this.$emit("change-page", page);
            },
            fetchSongs() {
                // Get all songs
                const url = "http://webservies.be/eurosong/Songs";

                fetch(url)
                    .then((response) => {
                        return response.json();
                    })
                    .then((songs) => {
                        //call fetchVotes method to add totalPoints to songs
                        this.fetchVotes(songs);
                        this.fetchArtists(songs);
                    })
            },

            fetchArtists(songs) {
                const url = "http://webservies.be/eurosong/Artists";
                fetch(url)
                    .then((response) => {
                        return response.json();
                    })
                    .then((artists) => {
                        songs.map(song => {
                            const a = artists.find((artist) => artist.id == song.artist);
                            song.artist = a.name;
                            return song;
                        });
                        this.songs = songs;
                    });
            },
            fetchVotes(songs) {
                // Get all votes
                const url = "http://webservies.be/eurosong/Votes";

                fetch(url)
                    .then((response) => {
                        // response is text, so convert to json
                        return response.json();
                    })
                    .then((votes) => {
                        // loop over array songs with map method and then add totalPoints to songs with the sum of all vote points
                        let allPoints = 0;
                        songs.map(song => {
                            // create totalPoints variable to get the sum of all vote points
                            song.totalPoints = 0;
                            //filter all the votes out where the vote songID is the same as the song id, because there are multiple votes for 1 song, then add up all points for 1 song and add it to that song's object in the JSON (totalPoints)
                            votes.filter((vote) => vote.songID == song.id).forEach(element => {
                                if (element.songID == song.id) {
                                    song.totalPoints += element.points
                                    allPoints += element.points
                                    }
                            });
                            return song;
                        });

                        songs.forEach(song => {
                            song.percentage = Number((song.totalPoints / allPoints * 100).toFixed(2));
                            console.log(song.percentage)
                        })  
                        // sort songs from best to worst and change data of songs, so everything will get rerenderd;
                        this.songs = songs.sort((a, b) => b.totalPoints - a.totalPoints );
                    });
            }
        }
    }
</script>