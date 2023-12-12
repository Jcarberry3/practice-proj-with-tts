<template>
<div class="main-container">
  <!-- Header -->
  <div class="header">
    <el-header class="content-header">
      Lyrics to Speech Converter
    </el-header>
    <el-popover
      popper-class="popover-text"
      placement="top-start"
      title="Lyrics to Speech Converter"
      :width="200"
      trigger="hover"
    >
      <template #reference>
        <i class="el-icon-info"></i>
      </template>
        <span>By entering an artist name and a track name in the search boxes and clicking the 'Submit' button the lyrics of the submitted track are returned, that can then be read aloud using the buttons at the bottom of the page</span>
    </el-popover>
  </div>
    <!-- Body -->
    <el-container class="content-container">
      <el-aside class="search-box">
        <div style="margin-bottom: 15px;">
          <span style="color: black; font-size: 150%;">Search</span>
          <form autocomplete="off">
            <div class="autocomplete">
              <!-- Artist Search -->
              <el-autocomplete
              class="input-field"
              v-model="artistName"
              :fetch-suggestions="getArtists"
              placeholder="Enter artist name here..."
              :trigger-on-focus="false"
              @select="handleSelect"
              alt="Enter artist name here"
              ></el-autocomplete>

              <!-- Track Search -->
              <el-autocomplete
              class="input-field"
              v-model="trackName"
              :fetch-suggestions="getTracks"
              placeholder="Enter track name here..."
              :trigger-on-focus="false"
              @select="handleSelect"
              alt="Enter track name here"
              ></el-autocomplete>
            </div>
          </form>
        </div>
        <!-- Submit Button -->
        <el-button type="primary" size="small" @click="submit" alt="click here to submit artist and song query">Submit</el-button>
      </el-aside>
      <el-main class="lyrics-display">{{ lyrics }}</el-main>
  </el-container>
  <!-- Speech Control Buttons -->
  <div class="speech-control-buttons">
    <div class="speech-button" @click="getSpeech">
      <i class="el-icon-video-play"></i>
    </diV>
    <div class="speech-button" @click="pauseSpeech">
      <i class="el-icon-video-pause"></i>
    </div>
    <div class="speech-button" @click="stopSpeech">
      <i class="el-icon-refresh-left"></i>
    </div>
  </div>
</div>
</template>

<script>

export default {
  data() {
    return {
      input: '',
      // Internal variables
      artistName: '',
      trackName: '',
      fullQuery: '',

      //ID's
      artistId: null,
      trackId: null,
      albumId: null,

      lyrics: "",
    }
  },
  methods: {
    submit() {
      this.fullQuery = this.artistName + " " + this.trackName;
      this.getLyrics();
    },
    getLyrics() { // The API calls to happi.dev is limited to 8,000 calls a month, so if it stops working, it might have gone over
      // axios init
      const axios = require('axios');

      // Search for the user inputted artist + song
      axios.get("https://api.happi.dev/v1/music", {
        headers: {
          "x-happi-key": "e81695FMejR6OfTGISYtnBSI8Y4F6leMUqNxoUwl6M0SinNjipbiCII1"
        },
        params: {
          "q": this.fullQuery,
          "limit": 10,
          "lyrics": true
        }
      })
      .then((res) => { 
        if(res.data.result.length < 1)
        {
          this.lyrics = "That track or artist could not be found. Please try again.";
          return;
        }
        console.log(res);
        this.artistId = res.data.result[0].id_artist;
        this.trackId = res.data.result[0].id_track;
        this.albumId = res.data.result[0].id_album;
        console.log(this.artistId);
        console.log(this.trackId);
        console.log(this.albumId);


        //use the ID's from the previous search to search for the song lyrics
        axios.get(`https://api.happi.dev/v1/music/artists/${this.artistId}/albums/${this.albumId}/tracks/${this.trackId}/lyrics`, {
          headers: {
            "x-happi-key": "e81695FMejR6OfTGISYtnBSI8Y4F6leMUqNxoUwl6M0SinNjipbiCII1"
          }
        })
        .then((res) => { 
          console.log(res);
          this.lyrics = res.data.result.lyrics;
        });
      });
    },
    getSpeech() {
      if(window.speechSynthesis.paused)
      {
        window.speechSynthesis.resume();
      }
      else{
        let msg = new SpeechSynthesisUtterance(this.lyrics);
        window.speechSynthesis.cancel();
        window.speechSynthesis.speak(msg);
      }
    },
    pauseSpeech() {
      if(window.speechSynthesis.speaking)
      {
        window.speechSynthesis.pause();
      }
    },
    stopSpeech() {
      window.speechSynthesis.cancel();
    },
    getArtists(artistName, cb) {
      console.log(artistName);
      let apiKey = '940a9bdeb87261051ce8a5e2bd763ab8';

      let axios = require('axios');

      axios.get("http://ws.audioscrobbler.com/2.0/?method=artist.search&format=json", {
        params: {
          limit: 10,
          artist: artistName,
          api_key: apiKey
        }
      })
      .then((res) => {
        let artistArray = [];
        let resultsArtists = res.data.results.artistmatches.artist;
        for(let i = 0; i < resultsArtists.length; i++)
        {
          artistArray.push({'value': resultsArtists[i].name});
        }
        cb(artistArray); 
      });      
    },
    getTracks(trackName, cb) {
      console.log(trackName);
      let apiKey = '940a9bdeb87261051ce8a5e2bd763ab8';

      let axios = require('axios');

      axios.get("http://ws.audioscrobbler.com/2.0/?method=track.search&format=json", {
        params: {
          limit: 10,
          track: trackName,
          artist: this.artistName,
          api_key: apiKey
        }
      })
      .then((res) => {
        console.log(this.artistName);
        let trackArray = [];
        let resultsTracks = res.data.results.trackmatches.track;
        for(let i = 0; i < resultsTracks.length; i++)
        {
          trackArray.push({'value': resultsTracks[i].name});
        }
        cb(trackArray);
      });
    }
  }
}
</script>

<style lang="less">
* { //CSS reset
  margin: 0;
  padding: 0;
  border-radius: 4px;
}

.el-popover {
  word-break: normal !important;
  width: 350px !important;
}

.main-container {
  display: flex;
  flex-direction: column;
  align-items: center;

  width: 100vw;
  height: calc(100vh - 60px);

  padding-top: 60px;

  background-color: #181818;

  .header {
    display: flex;
    flex-direction: row;
    justify-content: center;

    min-width: 500px;
  
    .content-header {
      position: relative;
      display: flex;
      align-items: center;
      justify-content: center;

      width: 70%;

      font-size: 150%;

      color: white;

      margin-bottom: 20px;

      background-color: rgb(53, 53, 53);      
    }

    .el-icon-info {
        position: absolute;
        top: 8%;
        right: 35.5%;

        margin-top: 20px;
        margin-left: 10px;
        padding: 0px;

        font-size: 150%;

        color: white;
      }
  }

  .content-container {
    width: 90%;
    max-height: 400px;

    background-color: rgb(53, 53, 53);

    .search-box {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;

      margin: 20px;
      padding: 20px;

      background-color: #f2f2f2;

      border: solid 1px transparent;
      box-shadow: 0 2px 4px rgba(0, 0, 0, .12), 0 0 6px rgba(0, 0, 0, .04);

      .autocomplete {
        position: relative;
        display: inline-block;
      }
      .autocomplete-items {
        position: absolute;

        border: 1px solid #d4d4d4;
        border-bottom: none;
        border-top: none;

        z-index: 99;
        top: 100%;
        left: 0;
        right: 0;

        div {
          cursor: pointer;

          border-bottom: 1px solid #d4d4d4;

          padding: 10px;

          background-color: #fff;
          &:hover {
          background-color: #e9e9e9;
          }
        }
      }
      
      .autocomplete-active {
        background-color: dodgerBlue !important;
        color: #ffffff;
      }

      .input-field {
        color: black;
        margin: 15px 5px 15px 5px;
      }
    }

    .lyrics-display {

      margin: 20px;
      padding: 20px;

      color: rgb(39, 38, 38);
      background-color: #f2f2f2;
      
      box-shadow: 0 2px 4px rgba(0, 0, 0, .12), 0 0 6px rgba(0, 0, 0, .04);
    }
  }

  #ttsiframe {
    width: 320px;
    height: 250px;
  }

  .speech-control-buttons {
    display: flex;
    flex-direction: row;

    height: 50px;

    background-color: #f2f2f2;

    border-radius: 5px;

    margin-top: 15px;

    .speech-button {
      font-size: 2.5rem;

      margin: 2.5px 10px 0px 10px;

      .el-icon-video-play {
        color: #353535;

        transition: 0.3s;

        &:hover {
          color: #409eff
        }
      }
      .el-icon-video-pause {
        color: #353535;

        transition: 0.3s;

        &:hover {
          color: #409eff
        }
      }

      .el-icon-refresh-left {
        color: #353535;

        transition: 0.3s;

        &:hover {
          color: #409eff
        }
      }
    }
  }
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
