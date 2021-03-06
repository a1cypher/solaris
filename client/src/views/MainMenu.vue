<template>
<div>
  <div class="container col-xs-12 col-sm-10 col-md-10 col-lg-6 pr-1 pl-1">
    <div class="row no-gutters pb-0 pt-1" v-if="user">
      <div class="col">
        <router-link to="/account/settings"><i class="fas fa-user mr-1"></i>{{user.username}}</router-link>
      </div>
      <div class="col-auto">
        <router-link v-if="user" :to="{ name: 'account-achievements', params: { userId: user._id }}"><i class="fas fa-medal mr-1"></i>Achievements</router-link>
        <router-link to="/codex" class="ml-3"><i class="fas fa-question mr-1"></i>Help</router-link>
        <a href="javascript:;" @click="logout" :disabled="isLoggingOut" class="ml-3"><i class="fas fa-sign-out-alt mr-1"></i>Log Out</a>
      </div>
    </div>
  </div>

  <view-container>
    <view-title title="Main Menu" :hideHomeButton="true"></view-title>

    <div class="row pb-0">
      <div class="col-sm-12 col-md-6 col-lg-5">
        <p>A space strategy game filled with conquest, betrayal and subterfuge.</p>
        <p>Welcome back<span v-if="user">, <span class="text-warning">{{user.username}}</span></span>!</p>
      </div>
      <div class="col-sm-12 col-md-6 col-lg-7">
        <!-- player quick stats -->
        <achievements v-if="achievements" v-bind:victories="achievements.victories" v-bind:rank="achievements.rank" v-bind:renown="achievements.renown"/>
        <loading-spinner :loading="!achievements"></loading-spinner>
      </div>
    </div>

    <div class="row pb-0">
      <div class="col-sm-12 col-md-4 col-lg-4">
        <div class="card bg-dark text-white" @click="routeToPath('/game/active-games')">
          <img class="card-img" :src="require('../assets/screenshots/home-1.png')" alt="View my games">
          <div class="card-img-overlay">
            <h5 class="card-title">My Games</h5>
          </div>
        </div>
      </div>
      <div class="col-sm-12 col-md-4 col-lg-4" @click="routeToPath('/game/list')">
        <div class="card bg-dark text-white">
          <img class="card-img" :src="require('../assets/screenshots/home-2.png')" alt="Join a game">
          <div class="card-img-overlay">
            <h5 class="card-title">Join Game</h5>
          </div>
        </div>
      </div>
      <div class="col-sm-12 col-md-4 col-lg-4">
        <div class="card bg-dark text-white" @click="routeToPath('/leaderboard')">
          <img class="card-img" :src="require('../assets/screenshots/home-3.png')" alt="Leaderboard">
          <div class="card-img-overlay">
            <h5 class="card-title">Leaderboard</h5>
          </div>
        </div>
      </div>
    </div>

    <div class="row pb-0">
      <div class="col d-none d-md-block">
        <h4>Top 10 Players</h4>
        <leaderboard-table if="leaderboard" :leaderboard="leaderboard"></leaderboard-table>
        <loading-spinner :loading="!leaderboard"></loading-spinner>
      </div>
    </div>
  </view-container>
</div>
</template>

<script>
import LoadingSpinnerVue from '../components/LoadingSpinner'
import router from '../router'
import authService from '../services/api/auth'
import userService from '../services/api/user'
import ViewContainer from '../components/ViewContainer'
import ViewTitle from '../components/ViewTitle'
import Achievements from '../components/game/player/Achievements'
import LeaderboardTable from '../components/game/menu/LeaderboardTable'

export default {
  components: {
    'loading-spinner': LoadingSpinnerVue,
    'view-container': ViewContainer,
    'view-title': ViewTitle,
    'achievements': Achievements,
    'leaderboard-table': LeaderboardTable
  },
  data () {
    return {
      user: null,
      achievements: null,
      leaderboard: null,
      isLoggingOut: false
    }
  },
  mounted () {
    this.loadAchievements()
    this.loadLeaderboard()
  },
  methods: {
    async logout () {
      this.isLoggingOut = true

      await authService.logout()

      this.$store.commit('clearUserId')

      this.isLoggingOut = false

      router.push({ name: 'home' })
    },
    async loadAchievements () {
      try {
        let response = await userService.getMyUserInfo()

        this.user = response.data
        this.achievements = response.data.achievements
      } catch (err) {
        console.error(err)
      }
    },
    async loadLeaderboard () {
      try {
        let response = await userService.getLeaderboard(10)

        this.leaderboard = response.data.leaderboard
      } catch (err) {
        console.error(err)
      }
    },
    routeToPath(path) {
      router.push(path)
    }
  }
}
</script>

<style scoped>
button {
  display: block;
}

.row {
  padding-bottom: 15px;
}

.card {
  max-height: 150px;
  margin-bottom: 1rem;
  cursor: pointer;
}

.card-img {
  object-fit: cover;
  max-height: 150px;
}

.card-img-overlay {
  padding: 0.5rem;
}

.card-title {
  background-color: #375a7f;
  padding: 0.25rem;
  display: inline-block;
  border-radius: 3px;
}
</style>
