<template>
  <div id="app"> 
    <h1 class="title">古代汉语搭配助手</h1>
    <div class="search-box">
      <input 
        v-model="searchWord" 
        type="search"
        placeholder="请输入单字或词语..." 
        class="search-input" 
        @keyup.enter="findCollocations" 
      />
      <button @click="findCollocations" class="search-button">搜索</button>
    </div>

    <div v-if="collocations.length === 0 && searchPerformed" class="no-results">
      无结果！
    </div>

    <div v-if="Object.keys(groupedCollocations).length > 0" class="results">
      <div v-for="(collocations, deprel) in groupedCollocations" :key="deprel" class="category">
        <h2>{{ deprel }}</h2>
        <table class="results-table">
          <thead>
            <tr>
              <th>Collocation</th>
              <th>Frequency</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(item, index) in collocations.slice(0, 5)" :key="item.partner">
              <td>{{ item.form }} - {{ item.partner }}</td>
              <td>{{ item.Frequency }}</td>
            </tr>
            <tr v-if="showMore[deprel]" v-for="(item, index) in collocations.slice(5)" :key="'more-' + item.partner">
              <td>{{ item.form }} - {{ item.partner }}</td>
              <td>{{ item.Frequency }}</td>
            </tr>
          </tbody>
        </table>
        <div v-if="collocations.length > 5" class="collocation-item">
          <span class="show-more" @click="toggleShowMore(deprel)">
            {{ showMore[deprel] ? '折叠结果' : '显示更多结果' }}
          </span>
        </div>
      </div>
    </div>

    <button v-if="searchPerformed && showScrollTopButton" @click="scrollToTop" class="scroll-top-button">
      ↑ 返回顶部
    </button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      searchWord: '',
      collocations: [],
      showMore: {},
      searchPerformed: false,
      showScrollTopButton: false // To control the visibility of the scroll-to-top button
    };
  },
  computed: {
    groupedCollocations() {
      const grouped = {};
      this.collocations.forEach(item => {
        if (!grouped[item.deprel]) {
          grouped[item.deprel] = [];
        }
        grouped[item.deprel].push(item);
      });
      for (let deprel in grouped) {
        grouped[deprel].sort((a, b) => b.Frequency - a.Frequency);
      }
      return grouped;
    }
  },
  methods: {
    async findCollocations() {
      const response = await fetch('/collocations.json');
      const data = await response.json();
      this.collocations = data.filter(item => 
        item.form === this.searchWord || item.partner === this.searchWord
      );
      this.searchPerformed = true;
      this.showScrollTopButton = false; // Hide scroll-to-top button initially
    },
    toggleShowMore(deprel) {
      this.showMore[deprel] = !this.showMore[deprel];
    },
    scrollToTop() {
      window.scrollTo({ top: 0, behavior: 'smooth' });
    }
  },
  mounted() {
    window.addEventListener('scroll', () => {
      if (window.scrollY > 200) {
        this.showScrollTopButton = true;
      } else {
        this.showScrollTopButton = false;
      }
    });
  }
};
</script>

<style>
html, body {
  height: 100%;
  margin: 0;
  padding: 0;
  font-family: Arial, sans-serif;
  background: linear-gradient(to left, #e0f7fa, #80deea); /* Light blue gradient background */
}

#app {
  min-width: 80%;
  max-height: 90%;
  overflow: visible;
  max-width: 90%; /* Ensure container does not exceed viewport width */
  margin: 50px auto;
  padding: 20px;
  background-color: #ffffff;
  border-radius: 8px;
  box-shadow: none; /* Removed any additional shadows */
  display: flex;
  flex-direction: column;
  align-items: center;
}

.title {
  text-align: center;
  margin-bottom: 20px;
  font-size: 24px;
  color: #333;
  width: 100%;
}

.search-box {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
  width: 100%;
}

.search-input {
  padding: 10px;
  width: 70%;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 4px 0 0 4px;
}

.search-button {
  padding: 10px 20px;
  font-size: 16px;
  background-color: #007BFF;
  color: #ffffff;
  border: none;
  cursor: pointer;
  border-radius: 0 4px 4px 0;
}

.search-button:hover {
  background-color: #0056b3;
}

.results {
  width: 100%;
  margin-top: 20px;
}

.category {
  margin-bottom: 30px;
  color: black;
}

.results-table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 10px;
}

.results-table th, .results-table td {
  border: 1px solid #ddd;
  padding: 10px;
  text-align: left;
  color: #000;
}

.results-table th {
  background-color: #f2f2f2;
}

.collocation-item {
  margin-bottom: 10px;
}

.show-more {
  color: #007BFF;
  cursor: pointer;
  font-size: 14px;
  display: inline-block;
  margin-top: 10px;
}

.show-more:hover {
  text-decoration: underline;
}

.no-results {
  text-align: center;
  font-size: 24px;
  font-weight: bold;
  color: black;
  margin-top: 20px;
}

.scroll-top-button {
  position: fixed;
  bottom: 30px;
  right: 30px;
  padding: 10px 20px;
  font-size: 16px;
  background-color: #007BFF;
  color: white;
  border: none;
  border-radius: 50px;
  cursor: pointer;
  box-shadow: 0px 2px 10px rgba(0, 0, 0, 0.3);
  z-index: 1000;
}

.scroll-top-button:hover {
  background-color: #0056b3;
}
</style>
