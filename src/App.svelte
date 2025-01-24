<script>
  import ValorHora from './components/ValorHora.svelte';
  import ListaJobs from './components/ListaJobs.svelte';
  import ResumoFinanceiro from './components/ResumoFinanceiro.svelte';
  import Configuracoes from './components/Configuracoes.svelte';
  import ExportarDados from './components/ExportarDados.svelte';
  import Categorias from './components/Categorias.svelte';

  let valorHora = 0;
  let jobs = [];
  let categorias = [
    { id: 1, nome: 'Website', cor: 'blue' },
    { id: 2, nome: 'Mobile App', cor: 'green' },
    { id: 3, nome: 'Design', cor: 'purple' },
    { id: 4, nome: 'Consultoria', cor: 'orange' }
  ];
  let configuracoes = {
    horasTrabalhoRegular: 8,
    diasTrabalhoRegular: 5,
    taxaUrgencia: 1.2,
    taxaComplexidade: 1.3,
    moeda: 'BRL',
    impostos: 0.06
  };

  let activeTab = 'Projetos';

  function carregarDados() {
    if (typeof window !== 'undefined') {
      const dadosSalvos = localStorage.getItem('jobscalc-dados');
      if (dadosSalvos) {
        try {
          const dados = JSON.parse(dadosSalvos);
          valorHora = dados.valorHora || 0;
          jobs = Array.isArray(dados.jobs) ? dados.jobs : [];
          configuracoes = dados.configuracoes || configuracoes;
          categorias = Array.isArray(dados.categorias) ? dados.categorias : categorias;
        } catch (error) {
          console.error('Erro ao carregar dados:', error);
          // Em caso de erro, mantém os valores padrão
        }
      }
    }
  }

  // Carregar dados quando o componente for montado
  import { onMount } from 'svelte';
  onMount(() => {
    try {
      carregarDados();
    } catch (error) {
      console.error('Erro ao carregar dados:', error);
    }
  });

  // Salvar dados no localStorage quando houver mudanças
  $: if (typeof window !== 'undefined') {
    try {
      localStorage.setItem('jobscalc-dados', JSON.stringify({
        valorHora,
        jobs,
        configuracoes,
        categorias
      }));
    } catch (error) {
      console.error('Erro ao salvar dados:', error);
    }
  }

  function handleJobAdded(event) {
    const job = event.detail;
    jobs = [...jobs, job];
  }

  function handleJobDeleted(event) {
    const jobId = event.detail;
    jobs = jobs.filter(job => job.id !== jobId);
  }

  function handleJobEdited(event) {
    const jobEditado = event.detail;
    jobs = jobs.map(job => job.id === jobEditado.id ? jobEditado : job);
  }

  function handleJobDuplicated(event) {
    const jobId = event.detail;
    const jobToDuplicate = jobs.find(job => job.id === jobId);
    if (jobToDuplicate) {
      const newJob = {
        ...jobToDuplicate,
        id: Date.now(),
        nome: `${jobToDuplicate.nome} (Cópia)`
      };
      jobs = [...jobs, newJob];
    }
  }

  function handleJobsUpdated(event) {
    jobs = event.detail;
  }

  function handleCategoriaAdded(event) {
    const categoria = event.detail;
    categorias = [...categorias, categoria];
  }

  function handleCategoriaEdited(event) {
    const editedCategoria = event.detail;
    categorias = categorias.map(c => c.id === editedCategoria.id ? editedCategoria : c);
  }

  function handleCategoriaDeleted(event) {
    const categoriaId = event.detail;
    categorias = categorias.filter(categoria => categoria.id !== categoriaId);
  }
</script>

<div class="min-h-screen bg-gray-900 text-gray-100">
  <header class="bg-gray-800 border-b border-gray-700">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-4">
      <h1 class="text-3xl font-bold text-transparent bg-clip-text bg-gradient-to-r from-indigo-400 to-indigo-600">
        Calculadora de Preço por Trabalho
      </h1>
      <p class="mt-1 text-gray-400">Calcule o valor dos seus projetos com base no seu preço por hora</p>
    </div>
  </header>

  <main class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
    <div class="flex gap-4 mb-6">
      {#each ['Projetos', 'Categorias', 'Configurações'] as tab}
        <button
          class="px-4 py-2 rounded-lg font-medium transition-colors {activeTab === tab ? 'bg-indigo-500 text-white' : 'text-gray-400 hover:text-gray-200'}"
          on:click={() => activeTab = tab}
        >
          {tab}
        </button>
      {/each}
    </div>

    {#if activeTab === 'Projetos'}
      <div class="grid grid-cols-1 xl:grid-cols-12 gap-8">
        <div class="xl:col-span-3 space-y-8">
          <ValorHora bind:valorHora={valorHora} />
          <ExportarDados {jobs} {valorHora} {configuracoes} />
        </div>
        
        <div class="xl:col-span-6">
          <ListaJobs
            {valorHora}
            {configuracoes}
            {categorias}
            {jobs}
            on:jobAdded={handleJobAdded}
            on:jobDeleted={handleJobDeleted}
            on:jobEdited={handleJobEdited}
            on:jobDuplicated={handleJobDuplicated}
            on:jobsUpdated={handleJobsUpdated}
          />
        </div>
        
        <div class="xl:col-span-3">
          <ResumoFinanceiro {jobs} {configuracoes} />
        </div>
      </div>
    {:else if activeTab === 'Categorias'}
      <div class="max-w-3xl mx-auto">
        <Categorias
          bind:categorias
          on:categoriaAdded={handleCategoriaAdded}
          on:categoriaEdited={handleCategoriaEdited}
          on:categoriaDeleted={handleCategoriaDeleted}
        />
      </div>
    {:else if activeTab === 'Configurações'}
      <div class="max-w-3xl mx-auto">
        <Configuracoes bind:configuracoes />
      </div>
    {/if}
  </main>

  <footer class="bg-gray-800 border-t border-gray-700 mt-auto">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-4">
      <p class="text-center text-gray-400 text-sm">
        Desenvolvido com 💜 por Levi Silvino
      </p>
    </div>
  </footer>
</div>
