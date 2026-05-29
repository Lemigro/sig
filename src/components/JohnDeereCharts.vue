<script setup>
import { onBeforeUnmount, onMounted } from 'vue';
import Chart from 'chart.js/auto';

const GR = '#367c2b';
const YL = '#ffde00';
const OR = '#d97706';
const RD = '#991b1b';
const GRID = 'rgba(0,0,0,0.05)';
const base = { responsive: true, maintainAspectRatio: false, plugins: { legend: { display: false } } };
const tk = { color: '#6b6460', font: { family: "'Outfit',sans-serif", size: 11 } };

let anualChart;
let wirtgenChart;
let produtosChart;
let filialChart;
let pandemiaChart;

onMounted(() => {
  anualChart = new Chart('cAnual', {
    type: 'bar',
    data: {
      labels: ['2019', '2020', '2021', '2022', '2023', '2024'],
      datasets: [{ data: [262, 330, 428, 627, 528, 478], backgroundColor: [GR, RD, GR, YL, GR, GR], borderRadius: 5, borderSkipped: false }]
    },
    options: {
      ...base,
      scales: {
        y: { ticks: { ...tk }, grid: { color: GRID }, border: { display: false } },
        x: { ticks: tk, grid: { display: false }, border: { display: false } }
      }
    }
  });

  wirtgenChart = new Chart('cWirtgen', {
    type: 'bar',
    data: {
      labels: ['2019', '2020', '2021', '2022', '2023', '2024'],
      datasets: [
        { label: 'John Deere', data: [100, 95, 93, 93, 78, 72], backgroundColor: GR, borderRadius: [0, 0, 0, 0], stack: 's' },
        { label: 'Wirtgen', data: [0, 5, 7, 7, 22, 28], backgroundColor: YL, borderRadius: [3, 3, 3, 3], stack: 's' }
      ]
    },
    options: {
      ...base,
      plugins: {
        legend: { display: true, labels: { color: '#6b6460', font: { family: "'Outfit',sans-serif", size: 11 }, boxWidth: 10, borderRadius: 2 } },
        tooltip: { callbacks: { label: (v) => `${v.dataset.label}: ${v.raw}%` } }
      },
      scales: {
        x: { ticks: tk, grid: { display: false }, border: { display: false } },
        y: { ticks: { ...tk, callback: (v) => `${v}%` }, grid: { color: GRID }, border: { display: false }, max: 100 }
      }
    }
  });

  produtosChart = new Chart('cProdutos', {
    type: 'bar',
    indexAxis: 'y',
    data: {
      labels: ['Retroescavadeira', 'Escavadeira', 'Pá Carregadeira', 'Compactador', 'Motoniveladora', 'Trator Esteira', 'Vibroacabadora', 'Fresadora', 'Recicladora'],
      datasets: [{ data: [1157, 606, 516, 332, 252, 238, 93, 42, 14], backgroundColor: [GR, GR, GR, YL, YL, YL, OR, OR, RD], borderRadius: 3 }]
    },
    options: {
      ...base,
      scales: {
        x: { ticks: tk, grid: { color: GRID }, border: { display: false } },
        y: { ticks: tk, grid: { display: false }, border: { display: false } }
      }
    }
  });

  const filData = {
    total: { l: ['REC', 'SSA', 'FOR', 'BAY', 'PET', 'LEM'], d: [863, 858, 414, 187, 186, 145] },
    2022: { l: ['REC', 'SSA', 'FOR', 'LEM', 'PET', 'BAY'], d: [207, 202, 85, 33, 32, 28] },
    2024: { l: ['REC', 'SSA', 'FOR', 'PET', 'BAY', 'LEM'], d: [130, 104, 51, 25, 21, 18] }
  };

  filialChart = new Chart('cFilial', {
    type: 'bar',
    indexAxis: 'y',
    data: { labels: filData.total.l, datasets: [{ data: filData.total.d, backgroundColor: GR, borderRadius: 3 }] },
    options: {
      ...base,
      scales: {
        x: { ticks: tk, grid: { color: GRID }, border: { display: false } },
        y: { ticks: tk, grid: { display: false }, border: { display: false } }
      }
    }
  });

  window.swFil = (k, el) => {
    document.querySelectorAll('#tabsFil .tab').forEach((t) => t.classList.remove('active'));
    el.classList.add('active');
    filialChart.data.labels = filData[k].l;
    filialChart.data.datasets[0].data = filData[k].d;
    filialChart.update();
  };

  pandemiaChart = new Chart('cPandemia', {
    type: 'line',
    data: {
      labels: ['Jan', 'Fev', 'Mar', 'Abr', 'Mai', 'Jun', 'Jul', 'Ago', 'Set', 'Out', 'Nov', 'Dez'],
      datasets: [{
        data: [22, 26, 25, 17, 18, 58, 10, 26, 24, 26, 33, 27],
        borderColor: RD,
        backgroundColor: 'rgba(153,27,27,0.06)',
        fill: true,
        tension: 0.3,
        pointRadius: 5,
        pointBackgroundColor: (ctx) => (ctx.dataIndex === 6 ? '#d94040' : RD),
        pointBorderWidth: (ctx) => (ctx.dataIndex === 6 ? 3 : 1)
      }]
    },
    options: {
      ...base,
      scales: {
        y: { ticks: tk, grid: { color: GRID }, border: { display: false } },
        x: { ticks: tk, grid: { display: false }, border: { display: false } }
      },
      plugins: {
        ...base.plugins,
        tooltip: { callbacks: { label: (v) => `${v.raw} vendas` } }
      }
    }
  });
});

onBeforeUnmount(() => {
  if (anualChart) anualChart.destroy();
  if (wirtgenChart) wirtgenChart.destroy();
  if (produtosChart) produtosChart.destroy();
  if (filialChart) filialChart.destroy();
  if (pandemiaChart) pandemiaChart.destroy();
  delete window.swFil;
});
</script>

<template></template>
