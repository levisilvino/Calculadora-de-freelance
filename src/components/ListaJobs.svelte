<script>
  export let valorHora;
  export let configuracoes;
  export let categorias;
  export let jobs = [];
  let novoJob = { 
    nome: '', 
    horasEstimadas: 0,
    urgente: false,
    complexo: false,
    descricao: '',
    categoriaId: '',
    status: 'a_fazer'
  };

  let showNovoJob = false;

  import { onMount } from 'svelte';
  import { createEventDispatcher } from 'svelte';
  import { slide } from 'svelte/transition';

  const dispatch = createEventDispatcher();

  const STATUS_OPTIONS = [
    { value: 'a_fazer', label: 'A Fazer', color: 'text-yellow-400' },
    { value: 'feito', label: 'Feito', color: 'text-green-400' },
    { value: 'pago', label: 'Pago', color: 'text-indigo-400' },
    { value: 'atrasado', label: 'Atrasado', color: 'text-red-400' }
  ];

  function getStatusColor(status) {
    const statusOption = STATUS_OPTIONS.find(opt => opt.value === status);
    return statusOption ? statusOption.color : 'text-gray-400';
  }

  function getStatusLabel(status) {
    const statusOption = STATUS_OPTIONS.find(opt => opt.value === status);
    return statusOption ? statusOption.label : 'Desconhecido';
  }

  function calcularValorJob(horas, urgente, complexo, valorHoraJob) {
    // Primeiro calcula o valor base com as taxas de urgência e complexidade
    let valorBase = horas * valorHoraJob;
    if (urgente) valorBase *= configuracoes.taxaUrgencia;
    if (complexo) valorBase *= configuracoes.taxaComplexidade;
    
    // Depois adiciona o imposto sobre o valor total
    let valorTotal = valorBase;
    if (configuracoes.impostos > 0) {
      const imposto = valorBase * configuracoes.impostos;
      valorTotal = valorBase + imposto;
      console.log('Calculando imposto:', {
        valorBase,
        taxaImposto: configuracoes.impostos,
        imposto,
        valorTotal
      });
    }
    
    return Number(valorTotal.toFixed(2));
  }

  function migrarJobsAntigos() {
    const jobsAtualizados = jobs.map(job => {
      if (!job.valorHora) {
        // Se não tem valorHora, recalcula com o valor original
        let valorBase = job.valorTotal;
        
        // Primeiro remove o imposto se houver
        if (configuracoes.impostos > 0) {
          valorBase = valorBase / (1 + configuracoes.impostos);
        }
        
        // Depois remove as taxas de urgência e complexidade
        if (job.complexo) valorBase /= configuracoes.taxaComplexidade;
        if (job.urgente) valorBase /= configuracoes.taxaUrgencia;
        
        // Por fim divide pelas horas para ter o valor hora
        valorBase = valorBase / job.horasEstimadas;
        
        return {
          ...job,
          valorHora: Number(valorBase.toFixed(2))
        };
      }
      return job;
    });

    // Se houver mudanças, dispara evento para atualizar
    if (JSON.stringify(jobsAtualizados) !== JSON.stringify(jobs)) {
      dispatch('jobsUpdated', jobsAtualizados);
    }
  }

  // Chamar migração quando o componente montar
  onMount(migrarJobsAntigos);

  // Chamar migração quando jobs ou configuracoes mudarem
  $: {
    if (jobs && configuracoes) {
      migrarJobsAntigos();
    }
  }

  function adicionarJob(e) {
    e.preventDefault();
    
    if (novoJob.nome && novoJob.horasEstimadas > 0) {
      const job = {
        id: Date.now(),
        nome: novoJob.nome,
        descricao: novoJob.descricao,
        horasEstimadas: novoJob.horasEstimadas,
        urgente: novoJob.urgente,
        complexo: novoJob.complexo,
        categoriaId: novoJob.categoriaId,
        status: novoJob.status,
        valorHora: valorHora,
        valorTotal: calcularValorJob(
          novoJob.horasEstimadas,
          novoJob.urgente,
          novoJob.complexo,
          valorHora
        )
      };
      dispatch('jobAdded', job);
      novoJob = { 
        nome: '', 
        horasEstimadas: 0,
        urgente: false,
        complexo: false,
        descricao: '',
        categoriaId: '',
        status: 'a_fazer'
      };
      showNovoJob = false;
    }
  }

  function editarJob(job) {
    editandoJob = { ...job };
  }

  function salvarEdicao() {
    if (editandoJob) {
      editandoJob.valorTotal = calcularValorJob(
        editandoJob.horasEstimadas,
        editandoJob.urgente,
        editandoJob.complexo,
        editandoJob.valorHora || valorHora
      );
      dispatch('jobEdited', editandoJob);
      editandoJob = null;
    }
  }

  function atualizarStatus(job, novoStatus) {
    const jobAtualizado = { ...job, status: novoStatus };
    dispatch('jobEdited', jobAtualizado);
  }

  function removerJob(id) {
    if (confirm('Tem certeza que deseja excluir este projeto?')) {
      dispatch('jobDeleted', id);
    }
  }

  function getCategoriaById(id) {
    return categorias.find(cat => cat.id === id);
  }

  function getCorClasse(cor) {
    const classes = {
      blue: 'bg-blue-100 text-blue-800',
      green: 'bg-green-100 text-green-800',
      purple: 'bg-purple-100 text-purple-800',
      orange: 'bg-orange-100 text-orange-800',
      red: 'bg-red-100 text-red-800',
      yellow: 'bg-yellow-100 text-yellow-800',
      indigo: 'bg-indigo-100 text-indigo-800',
      pink: 'bg-pink-100 text-pink-800'
    };
    return classes[cor] || classes.blue;
  }

  let filtroCategoria = null; // Categoria selecionada para filtro
  
  // Agrupar jobs por categoria
  $: jobsPorCategoria = categorias.map(categoria => {
    const jobsDaCategoria = jobs.filter(job => job.categoriaId === categoria.id);
    const valorTotal = jobsDaCategoria.reduce((total, job) => total + job.valorTotal, 0);
    return {
      ...categoria,
      jobs: jobsDaCategoria,
      valorTotal
    };
  });

  // Filtrar jobs baseado na categoria selecionada
  $: jobsFiltrados = filtroCategoria 
    ? jobs.filter(job => job.categoriaId === filtroCategoria)
    : jobs;

  $: valorTotal = jobsFiltrados.reduce((total, job) => total + job.valorTotal, 0);
</script>

<div class="bg-gray-800 rounded-lg shadow-md p-6">
  <div class="flex items-center justify-between mb-6">
    <div>
      <h2 class="text-2xl font-semibold mb-6 text-gray-200">Projetos (Jobs)</h2>
    </div>
    <button
      class="px-4 py-2 bg-gradient-to-r from-indigo-500 to-indigo-600 text-white rounded-lg font-medium hover:from-indigo-600 hover:to-indigo-700 transition-colors flex items-center gap-2"
      on:click={() => showNovoJob = !showNovoJob}
    >
      <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
        {#if showNovoJob}
          <path fill-rule="evenodd" d="M3 10a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1z" clip-rule="evenodd" />
        {:else}
          <path fill-rule="evenodd" d="M10 3a1 1 0 011 1v5h5a1 1 0 110 2h-5v5a1 1 0 11-2 0v-5H4a1 1 0 110-2h5V4a1 1 0 011-1z" clip-rule="evenodd" />
        {/if}
      </svg>
      {showNovoJob ? 'Cancelar' : 'Novo Projeto'}
    </button>
  </div>

  <!-- Categorias como filtros -->
  <div class="flex flex-wrap gap-2 mb-6">
    <button
      class="px-3 py-1 rounded-lg text-sm font-medium transition-colors {!filtroCategoria ? 'bg-indigo-500 text-white' : 'bg-gray-700 text-gray-300 hover:bg-gray-600'}"
      on:click={() => filtroCategoria = null}
    >
      Todos
    </button>
    {#each jobsPorCategoria as categoria}
      {#if categoria.jobs.length > 0}
        <button
          class="px-3 py-1 rounded-lg text-sm font-medium transition-colors {filtroCategoria === categoria.id ? 'bg-indigo-500 text-white' : 'bg-gray-700 text-gray-300 hover:bg-gray-600'}"
          on:click={() => filtroCategoria = categoria.id}
        >
          {categoria.nome}
          <span class="ml-2 text-xs opacity-75">
            {new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(categoria.valorTotal)}
          </span>
        </button>
      {/if}
    {/each}
  </div>

  {#if showNovoJob}
    <div class="bg-gray-700/50 rounded-xl p-6 border border-gray-600 mb-6" in:slide={{ duration: 200 }} out:slide={{ duration: 200 }}>
      <form class="space-y-4" on:submit|preventDefault={adicionarJob}>
        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
          <div>
            <label for="nome-job" class="block text-sm font-medium text-gray-300 mb-1">
              Nome do Projeto
            </label>
            <input 
              type="text" 
              id="nome-job"
              bind:value={novoJob.nome}
              class="w-full px-3 py-2 bg-gray-700 border border-gray-600 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 text-gray-100"
              placeholder="Ex: Website E-commerce"
              required
            >
          </div>

          <div>
            <label for="categoria" class="block text-sm font-medium text-gray-300 mb-1">
              Categoria
            </label>
            <select 
              id="categoria"
              bind:value={novoJob.categoriaId}
              class="w-full px-3 py-2 bg-gray-700 border border-gray-600 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 text-gray-100"
            >
              <option value="">Selecione uma categoria</option>
              {#each categorias as categoria}
                <option value={categoria.id}>{categoria.nome}</option>
              {/each}
            </select>
          </div>
        </div>

        <div>
          <label for="descricao-job" class="block text-sm font-medium text-gray-300 mb-1">
            Descrição do Projeto
          </label>
          <textarea 
            id="descricao-job"
            bind:value={novoJob.descricao}
            class="w-full px-3 py-2 bg-gray-700 border border-gray-600 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 text-gray-100"
            rows="3"
            placeholder="Descreva os detalhes do projeto..."
          ></textarea>
        </div>

        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
          <div>
            <label for="horas-estimadas" class="block text-sm font-medium text-gray-300 mb-1">
              Horas Estimadas
            </label>
            <div class="relative">
              <input 
                type="number" 
                id="horas-estimadas"
                bind:value={novoJob.horasEstimadas}
                class="w-full px-3 py-2 bg-gray-700 border border-gray-600 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 text-gray-100"
                min="1"
                step="0.5"
                required
              >
              <div class="absolute inset-y-0 right-0 pr-3 flex items-center pointer-events-none">
                <span class="text-gray-400">horas</span>
              </div>
            </div>
          </div>

          <div class="flex items-center space-x-6">
            <label class="flex items-center">
              <input 
                type="checkbox"
                bind:checked={novoJob.urgente}
                class="rounded border-gray-600 text-indigo-500 focus:ring-indigo-500"
              >
              <span class="ml-2 text-sm text-gray-300">Projeto Urgente (+{((configuracoes.taxaUrgencia - 1) * 100).toFixed()}%)</span>
            </label>
            <label class="flex items-center">
              <input 
                type="checkbox"
                bind:checked={novoJob.complexo}
                class="rounded border-gray-600 text-indigo-500 focus:ring-indigo-500"
              >
              <span class="ml-2 text-sm text-gray-300">Projeto Complexo (+{((configuracoes.taxaComplexidade - 1) * 100).toFixed()}%)</span>
            </label>
          </div>
        </div>

        <div>
          <label for="status" class="block text-sm font-medium text-gray-300 mb-1">
            Status
          </label>
          <select 
            id="status"
            bind:value={novoJob.status}
            class="w-full px-3 py-2 bg-gray-700 border border-gray-600 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 text-gray-100"
          >
            {#each STATUS_OPTIONS as statusOption}
              <option value={statusOption.value}>{statusOption.label}</option>
            {/each}
          </select>
        </div>

        <div class="pt-4">
          <button 
            type="submit"
            class="w-full px-4 py-2 bg-gradient-to-r from-indigo-500 to-indigo-600 text-white rounded-lg font-medium hover:from-indigo-600 hover:to-indigo-700 transition-colors"
          >
            Adicionar Projeto
          </button>
        </div>
      </form>
    </div>
  {/if}

  <div class="space-y-4">
    {#if jobsFiltrados.length === 0}
      <p class="text-center text-gray-400">Nenhum projeto cadastrado{filtroCategoria ? ' nesta categoria' : ''}</p>
    {:else}
      {#each jobsFiltrados as job (job.id)}
        <div class="p-4 bg-gray-700 rounded-lg">
          <div class="flex items-center justify-between mb-2">
            <div>
              <h3 class="text-lg font-medium text-gray-200">{job.nome}</h3>
              {#if job.descricao}
                <p class="text-sm text-gray-400 mt-1">{job.descricao}</p>
              {/if}
              
              <div class="text-sm text-gray-400">
                <p>
                  {job.horasEstimadas} horas × 
                  {#if job.valorHora !== undefined}
                    R$ {job.valorHora.toFixed(2)}
                  {:else}
                    R$ {(job.valorTotal / job.horasEstimadas).toFixed(2)}
                  {/if}
                </p>
                {#if job.urgente || job.complexo}
                  <p class="text-xs mt-1">
                    {#if job.urgente}
                      <span class="text-yellow-400">+{((configuracoes.taxaUrgencia - 1) * 100).toFixed()}% urgente</span>
                    {/if}
                    {#if job.urgente && job.complexo} • {/if}
                    {#if job.complexo}
                      <span class="text-blue-400">+{((configuracoes.taxaComplexidade - 1) * 100).toFixed()}% complexo</span>
                    {/if}
                  </p>
                {/if}
                <p class="font-medium text-gray-200 mt-2">
                  Total: {new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(job.valorTotal)}
                </p>
                <p class="text-sm {getStatusColor(job.status)} mt-2">
                  Status: {getStatusLabel(job.status)}
                </p>
              </div>
            </div>
            <div class="flex items-center space-x-2">
              <button 
                class="text-red-500 hover:text-red-700 transition-colors"
                on:click={() => removerJob(job.id)}
                aria-label="Remover {job.nome}"
              >
                Remover
              </button>
              <select 
                class="px-3 py-2 bg-gray-700 border border-gray-600 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 text-gray-100"
                on:change={(e) => atualizarStatus(job, e.target.value)}
              >
                {#each STATUS_OPTIONS as statusOption}
                  <option value={statusOption.value} selected={job.status === statusOption.value}>{statusOption.label}</option>
                {/each}
              </select>
            </div>
          </div>
        </div>
      {/each}

      <div class="pt-4 border-t border-gray-600">
        <p class="text-lg font-medium text-gray-200">
          Valor Total dos Projetos: R$ {valorTotal.toFixed(2)}
        </p>
      </div>
    {/if}
  </div>
</div>
