<template>
  <div :class="[$style.homePage]">
    <div class="header">
      <h1>FC Online Match Statistics Viewer</h1>
      <div class="search-bar">
        <input
          type="text"
          v-model="nicknameInput"
          placeholder="Enter nickname"
          @keyup.enter="search"
        />
        <button @click="search">Search</button>
      </div>
    </div>
    <div v-if="nicknameInput" class="results-header">
      <p>Showing results for: {{ nicknameInput }}</p>
    </div>
    <div v-if="matchDetails.length > 0" class="match-list">
      <div v-for="(matchDetailEntity, index) in matchDetails" :key="index" class="match-item">
        <div class="match-summary">
          <span>{{ matchDetailEntity.matchInfo[1].nickname }}</span>
          <span>{{ formatDate(matchDetailEntity.matchDate) }}</span>
          <span>{{ matchDetailEntity.matchInfo[0].matchDetail.matchResult }}</span>
          <button @click="toggleMatchExpansion('matchDetail', index)">
            <span v-if="isMatchExpanded === index">-</span>
            <span v-else>+</span>
          </button>
        </div>
        <div v-if="isMatchExpanded === index" class="match-details">
          <table>
            <tbody>
            <!--            Detail Section-->
            <div class="section-header">
              <button @click="toggleDetails('isMatchDetailExpanded', index)">
                <span
                  v-if="expandedSections['isMatchDetailExpanded'] === index">Hide Match Details</span>
                <span v-else>Show Match Details</span>
              </button>
            </div>
            <div v-if="expandedSections['isMatchDetailExpanded'] === index" class="section-details">
              <!-- Render shoot details here -->
              <!-- Example: -->
              <thead>
              <tr>
                <th>분류</th>
                <th>Me</th>
                <th>Opponent</th>
              </tr>
              <tr>
                <td>드리블 횟수</td>
                <td>{{ matchDetailEntity.matchInfo[0].matchDetail.dribble }}</td>
                <td>{{ matchDetailEntity.matchInfo[1].matchDetail.dribble }}</td>
              </tr>
              <tr>
                <td>코너킥</td>
                <td>{{ matchDetailEntity.matchInfo[0].matchDetail.cornerKick }}</td>
                <td>{{ matchDetailEntity.matchInfo[1].matchDetail.cornerKick }}</td>
              </tr>
              <tr>
                <td>오프사이드</td>
                <td>{{ matchDetailEntity.matchInfo[0].matchDetail.offsideCount }}</td>
                <td>{{ matchDetailEntity.matchInfo[1].matchDetail.offsideCount }}</td>
              </tr>
              </thead>
            </div>
            <!-- Shoot Section -->
            <div class="section-header">
              <button @click="toggleDetails('shoot', index)">
                <span v-if="expandedSections['isShootExpanded'] === index">Hide Shoot Details</span>
                <span v-else>Show Shoot Details</span>
              </button>
            </div>
            <div v-if="expandedSections['isShootExpanded'] === index" class="section-details">
              <!-- Render shoot details here -->
              <!-- Example: -->
              <p>Shoot Total: {{ matchDetailEntity.shoot.shootTotal }}</p>
              <p>Goal Total: {{ matchDetailEntity.shoot.goalTotal }}</p>
            </div>

            <!-- Pass Section -->
            <div class="section-header">
              <button @click="toggleDetails('pass', index)">
                <span v-if="expandedSections['isPassExpanded'] === index">Hide Pass Details</span>
                <span v-else>Show Pass Details</span>
              </button>
            </div>
            <div v-if="expandedSections['isPassExpanded'] === index" class="section-details">
              <!-- Render pass details here -->
              <!-- Example: -->
              <p>Pass Try: {{ matchDetailEntity.pass.passTry }}</p>
              <p>Pass Success: {{ matchDetailEntity.pass.passSuccess }}</p>
            </div>

            <!-- Defence Section -->
            <div class="section-header">
              <button @click="toggleDetails('defence', index)">
                <span
                  v-if="expandedSections['isDefenceExpanded'] === index">Hide Defence Details</span>
                <span v-else>Show Defence Details</span>
              </button>
            </div>
            <div v-if="expandedSections['isDefenceExpanded'] === index" class="section-details">
              <!-- Render defence details here -->
              <!-- Example: -->
              <p>Block Try: {{ matchDetailEntity.defence.blockTry }}</p>
              <p>Block Success: {{ matchDetailEntity.defence.blockSuccess }}</p>
            </div>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      nicknameInput: "",
      matchDetails: [],
      isMatchExpanded: null,
      expandedSections: {
        isMatchDetailExpanded: null,
        isShootExpanded: null,
        isPassExpanded: null,
        isDefenceExpanded: null,
      },
    };
  },
  methods: {
    async search() {
      if (this.nicknameInput) {
        try {
          const response = await fetch(`http://localhost:8000/match-statistics?nickname=${this.nicknameInput}`, {
            method: 'GET'
          });
          if (response.ok) {
            const data = await response.json();
            this.matchDetails = data.matchDetailDTO; // Assuming the API returns an object with a "matchInfo" array
          } else {
            console.error("Error:", response.statusText);
          }
        } catch (error) {
          console.error("Error:", error);
        }
      } else {
        alert("Please enter a nickname.");
      }
    },
    toggleMatchExpansion(section, index) {
      if (this.isMatchExpanded === index) {
        this.isMatchExpanded = null; // Collapse if already expanded
      } else {
        this.isMatchExpanded = index; // Expand
      }
    },
    toggleDetails(section, index) {
      if (this.expandedSections[section] === index) {
        this.expandedSections[section] = null; // Collapse if already expanded
      } else {
        this.expandedSections[section] = index; // Expand
      }
    },
    formatDate(dateString) {
      const date = new Date(dateString);
      return `${date.toLocaleDateString()} ${date.toLocaleTimeString()}`;
    },
  }
};
</script>

<style module lang="scss">
/* Your existing styles */
.section-header {
  margin-top: 10px;
}

.section-details {
  padding: 10px;
  border: 1px solid #ccc;
  background-color: #f5f5f5;
}
</style>
