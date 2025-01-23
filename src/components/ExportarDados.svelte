<script>
  export let jobs;
  export let valorHora;
  export let configuracoes;

  let previewAberto = false;
  let formatoSelecionado = 'json';

  function exportarCSV() {
    const headers = ['Nome do Projeto', 'Descrição', 'Horas Estimadas', 'Urgente', 'Complexo', 'Valor Total'];
    const rows = jobs.map(job => [
      job.nome,
      job.descricao,
      job.horasEstimadas,
      job.urgente ? 'Sim' : 'Não',
      job.complexo ? 'Sim' : 'Não',
      job.valorTotal.toFixed(2)
    ]);

    const csvContent = [
      headers.join(','),
      ...rows.map(row => row.map(cell => `"${cell}"`).join(','))
    ].join('\n');

    const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
    const link = document.createElement('a');
    const url = URL.createObjectURL(blob);
    link.setAttribute('href', url);
    link.setAttribute('download', 'jobscalc-projetos.csv');
    link.style.display = 'none';
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
  }

  function exportarJSON() {
    const dados = {
      configuracoes,
      valorHora,
      jobs,
      dataExportacao: new Date().toISOString(),
      resumo: {
        totalProjetos: jobs.length,
        totalHoras: jobs.reduce((total, job) => total + job.horasEstimadas, 0),
        valorTotal: jobs.reduce((total, job) => total + job.valorTotal, 0)
      }
    };

    const blob = new Blob([JSON.stringify(dados, null, 2)], { type: 'application/json' });
    const link = document.createElement('a');
    const url = URL.createObjectURL(blob);
    link.setAttribute('href', url);
    link.setAttribute('download', 'jobscalc-dados.json');
    link.style.display = 'none';
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
  }

  $: previewDados = formatoSelecionado === 'json' ? 
    {
      configuracoes,
      valorHora,
      jobs: jobs.slice(0, 2),
      dataExportacao: new Date().toISOString(),
      resumo: {
        totalProjetos: jobs.length,
        totalHoras: jobs.reduce((total, job) => total + job.horasEstimadas, 0),
        valorTotal: jobs.reduce((total, job) => total + job.valorTotal, 0)
      }
    } :
    {
      headers: ['Nome do Projeto', 'Horas Estimadas', 'Valor Total'],
      rows: jobs.slice(0, 2).map(job => [
        job.nome,
        job.horasEstimadas,
        job.valorTotal.toFixed(2)
      ])
    };
</script>

<div class="space-y-6">
  <div class="flex items-center justify-between">
    <div class="flex-1">
      <div class="text-2xl font-bold text-transparent bg-clip-text bg-gradient-to-r from-indigo-400 to-indigo-600">
        Backup & Exportação
      </div>
      <div class="text-sm text-gray-400 mt-1">
        Exporte seus dados para análise externa
      </div>
    </div>
    <div class="w-12 h-12 bg-gradient-to-br from-indigo-500 to-indigo-600 rounded-2xl flex items-center justify-center shadow-lg">
      <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-white" viewBox="0 0 20 20" fill="currentColor">
        <path fill-rule="evenodd" d="M3 17a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zm3.293-7.707a1 1 0 011.414 0L9 10.586V3a1 1 0 112 0v7.586l1.293-1.293a1 1 0 111.414 1.414l-3 3a1 1 0 01-1.414 0l-3-3a1 1 0 010-1.414z" clip-rule="evenodd" />
      </svg>
    </div>
  </div>

  <div class="bg-gray-700/50 rounded-xl p-4 border border-gray-600">
    <div class="flex gap-4">
      <label class="flex items-center">
        <input 
          type="radio" 
          bind:group={formatoSelecionado} 
          value="json" 
          class="text-indigo-500 border-gray-600 focus:ring-indigo-500"
        >
        <span class="ml-2 text-gray-300">JSON</span>
      </label>
      <label class="flex items-center">
        <input 
          type="radio" 
          bind:group={formatoSelecionado} 
          value="csv" 
          class="text-indigo-500 border-gray-600 focus:ring-indigo-500"
        >
        <span class="ml-2 text-gray-300">CSV</span>
      </label>
    </div>
  </div>

  <button
    class="w-full px-4 py-2 bg-gray-700/50 text-gray-300 border border-gray-600 rounded-xl font-medium hover:bg-gray-600 transition-colors flex items-center justify-center gap-2"
    on:click={() => previewAberto = !previewAberto}
  >
    <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
      {#if previewAberto}
        <path fill-rule="evenodd" d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z" clip-rule="evenodd" />
      {:else}
        <path fill-rule="evenodd" d="M14.707 12.707a1 1 0 01-1.414 0L10 9.414l-3.293 3.293a1 1 0 01-1.414-1.414l4-4a1 1 0 011.414 0l4 4a1 1 0 010 1.414z" clip-rule="evenodd" />
      {/if}
    </svg>
    {previewAberto ? 'Ocultar Preview' : 'Mostrar Preview'}
  </button>

  {#if previewAberto}
    <div class="bg-gray-800 rounded-xl p-4 border border-gray-700 overflow-auto max-h-64">
      <div class="flex items-center justify-between mb-3">
        <h3 class="text-sm font-medium text-gray-300">Preview ({formatoSelecionado.toUpperCase()})</h3>
        <span class="text-xs text-gray-500">Mostrando {jobs.length > 2 ? '2 de ' + jobs.length : jobs.length} projetos</span>
      </div>
      {#if formatoSelecionado === 'json'}
        <pre class="text-xs text-gray-400 font-mono whitespace-pre-wrap bg-gray-900 rounded-lg p-3">{JSON.stringify(previewDados, null, 2)}</pre>
      {:else}
        <div class="text-xs text-gray-400">
          <div class="grid grid-cols-3 gap-4 font-medium mb-2 text-gray-300 bg-gray-900 p-2 rounded-t-lg">
            {#each previewDados.headers as header}
              <div>{header}</div>
            {/each}
          </div>
          {#each previewDados.rows as row}
            <div class="grid grid-cols-3 gap-4 border-t border-gray-700 py-2">
              {#each row as cell}
                <div>{cell}</div>
              {/each}
            </div>
          {/each}
          {#if jobs.length > 2}
            <div class="text-gray-500 mt-2 italic text-center">... mais {jobs.length - 2} projetos ...</div>
          {/if}
        </div>
      {/if}
    </div>
  {/if}

  <div class="space-y-3">
    <button
      on:click={formatoSelecionado === 'json' ? exportarJSON : exportarCSV}
      class="w-full px-4 py-3 bg-gradient-to-r from-indigo-500 to-indigo-600 text-white rounded-xl font-medium hover:from-indigo-600 hover:to-indigo-700 transition-colors flex items-center justify-center gap-2 shadow-lg"
    >
      <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
        <path fill-rule="evenodd" d="M3 17a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zm3.293-7.707a1 1 0 011.414 0L9 10.586V3a1 1 0 112 0v7.586l1.293-1.293a1 1 0 111.414 1.414l-3 3a1 1 0 01-1.414 0l-3-3a1 1 0 010-1.414z" clip-rule="evenodd" />
      </svg>
      Exportar {formatoSelecionado.toUpperCase()}
    </button>

    <div class="text-xs text-gray-500 text-center">
      {#if jobs.length === 0}
        Nenhum projeto para exportar
      {:else}
        {jobs.length} projeto{jobs.length === 1 ? '' : 's'} disponível{jobs.length === 1 ? '' : 'is'} para exportação
      {/if}
    </div>
  </div>
</div>
