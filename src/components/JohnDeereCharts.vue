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
let sazonalChart;

const meses = ['Jan', 'Fev', 'Mar', 'Abr', 'Mai', 'Jun', 'Jul', 'Ago', 'Set', 'Out', 'Nov', 'Dez'];

// Vendas anuais / mix JD·Wirtgen: contagem por ano na base v03 (docs PDF). 2025 = 522 NFs, Wirtgen 29%.
// Faturamento mensal R$ mi — Relatório 9 (sazonalidade)
const sazData = {
  2019: [10.9, 5.7, 7.7, 6.0, 7.1, 8.1, 6.5, 12.4, 12.0, 11.4, 6.0, 8.5],
  2020: [9.9, 12.3, 11.0, 7.0, 8.1, 25.4, 4.6, 14.0, 14.5, 16.5, 13.1, 16.4],
  2022: [18.5, 31.0, 47.9, 34.6, 52.3, 56.9, 72.1, 66.5, 57.2, 48.2, 35.5, 37.5],
  2023: [43.0, 35.4, 61.2, 27.6, 41.7, 33.2, 52.7, 36.6, 41.9, 60.4, 51.7, 45.5],
  2024: [42.0, 34.4, 43.1, 44.6, 56.9, 50.0, 35.9, 45.9, 62.7, 41.5, 0, 0],
  2025: [48.3, 39.5, 49.6, 51.3, 65.4, 57.5, 41.3, 52.7, 72.1, 47.8, 0, 0]
};

const weakMonths = new Set([0, 6]);

onMounted(() => {
  anualChart = new Chart('cAnual', {
    type: 'bar',
    data: {
      labels: ['2019', '2020', '2021', '2022', '2023', '2024', '2025'],
      datasets: [{ data: [262, 330, 428, 627, 528, 478, 522], backgroundColor: [GR, RD, GR, YL, GR, GR, GR], borderRadius: 5, borderSkipped: false }]
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
      labels: ['2019', '2020', '2021', '2022', '2023', '2024', '2025'],
      datasets: [
        { label: 'John Deere', data: [100, 95, 93, 93, 78, 72, 71], backgroundColor: GR, borderRadius: [0, 0, 0, 0], stack: 's' },
        { label: 'Wirtgen', data: [0, 5, 7, 7, 22, 28, 29], backgroundColor: YL, borderRadius: [3, 3, 3, 3], stack: 's' }
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

  function sazColors(data) {
    const peak = Math.max(...data);
    return data.map((v, i) => {
      if (weakMonths.has(i)) return OR;
      if (v === peak && v > 0) return YL;
      return GR;
    });
  }

  sazonalChart = new Chart('cSazonal', {
    type: 'bar',
    data: {
      labels: meses,
      datasets: [{
        data: sazData[2022],
        backgroundColor: sazColors(sazData[2022]),
        borderRadius: 4
      }]
    },
    options: {
      ...base,
      plugins: {
        ...base.plugins,
        tooltip: { callbacks: { label: (v) => `R$ ${v.raw} mi` } }
      },
      scales: {
        y: { ticks: { ...tk, callback: (v) => `${v} mi` }, grid: { color: GRID }, border: { display: false } },
        x: { ticks: tk, grid: { display: false }, border: { display: false } }
      }
    }
  });

  window.swSaz = (yr, el) => {
    document.querySelectorAll('#tabsSaz .tab').forEach((t) => t.classList.remove('active'));
    el.classList.add('active');
    const data = sazData[yr];
    sazonalChart.data.datasets[0].data = data;
    sazonalChart.data.datasets[0].backgroundColor = sazColors(data);
    sazonalChart.update();
  };
});

onBeforeUnmount(() => {
  if (anualChart) anualChart.destroy();
  if (wirtgenChart) wirtgenChart.destroy();
  if (produtosChart) produtosChart.destroy();
  if (filialChart) filialChart.destroy();
  if (pandemiaChart) pandemiaChart.destroy();
  if (sazonalChart) sazonalChart.destroy();
  delete window.swFil;
  delete window.swSaz;
});
</script>

<template></template>
