<script setup>
import { ref, shallowRef, computed, watch, nextTick } from "vue";
import Chart from "chart.js/auto";

const problems = ref([]);

const problemChartElement = ref(null); // chart element
const problemChart = shallowRef(null); // chart instance

const problemInput = ref(0);
const currProblem = computed(() => {
  return problems.value.sort((a, b) => b.date - a.date)[0] || { problem: 0 };
});

const addProblem = () => {
  problems.value.push({
    problem: problemInput.value,
    date: new Date().getTime(),
  });
};





// watch(
//   problems,
//   () => {
//     if (problemChart.value) {
//       problemChart.value.data.labels = problems.value.map((p) =>
//         new Date(p.date).toLocaleDateString()
//       );
//       problemChart.value.data.datasets[0].data = problems.value.map(
//         (p) => p.problem
//       );
//       problemChart.value.update();
//     }
//   },
//   { immediate: true }
// );

watch(problems, (newProblems) => {
	const ws = [...newProblems]

	if (problemChart.value) {
		problemChart.value.data.labels = ws
			.sort((a, b) => a.date - b.date)
			.map(problem => new Date(problem.date).toLocaleDateString() )
			.slice(-7)

		problemChart.value.data.datasets[0].data = ws
			.sort((a, b) => a.date - b.date)
			.map(problem => problem.problem)
			.slice(-7)

		problemChart.value.update()
		return
	}

	nextTick(() => {
		problemChart.value = new Chart(problemChartElement.value.getContext('2d'), {
			type: 'line',
			data: {
				labels: ws
					.sort((a, b) => a.date - b.date)
					.map(problem => new Date(problem.date).toLocaleDateString()),
				datasets: [
					{
						label: 'Problem',
						data: ws
							.sort((a, b) => a.date - b.date)
							.map(problem => problem.problem),
						backgroundColor: 'rgba(255, 105, 180, 0.2)',
						borderColor: 'rgba(255, 105, 180, 1)',
						borderWidth: 1,
						fill: true
					}
				]
			},
			options: {
				responsive: true,
				maintainAspectRatio: false
			}
		})
	})
}, { deep: true })


</script>

<template>
  <main>
    <h2>Problem Tracker</h2>
    <div class="current">
      <small> You Solved</small> <span>{{ currProblem.problem }}</span>
      <small> Problems Today</small>
    </div>
    <form @submit.prevent="addProblem">
      <input type="number" step="1" v-model="problemInput" />
      <input type="submit" value="Add Problem" />
    </form>
    <div v-if="problems && problems.length > 0">
      <h3>Last 7 Days</h3>
       <div class="canvas-box">
        <canvas ref="problemChartElement" ></canvas>
      </div>

      <div class="problem-list">
        <h3>Problem List</h3>
        <ul>
          <li v-for="problem in problems">
            <span>
              {{ problem.problem }} <small> Problems</small>
            </span>
          </li>
        </ul>
      </div> 
    </div>
  </main>
</template>

<style scoped>
main {
  max-width: 600px;
  margin: 0 auto;
}

h2 {
  text-align: center;
  margin-bottom: 20px;
}

.current {
  text-align: center;
  margin: 20px 0;
}

form {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
}

input[type="number"] {
  padding: 8px;
  margin-right: 8px;
  font-size: 16px;
}

canvas {
  width: 100%;
  height: 300px; /* Set an appropriate height for your chart */
}

.problem-list {
  margin-top: 20px;
}

ul {
  list-style: none;
  padding: 0;
}

li {
  margin: 8px 0;
  padding: 8px;
  border: 1px solid #ccc;
  display: flex;
  justify-content: space-between;
}

.problem-list h3 {
  text-align: center;
}

</style>
