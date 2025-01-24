<script>
  import ValorHora from './components/ValorHora.svelte';
  import ListaJobs from './components/ListaJobs.svelte';
  import ResumoFinanceiro from './components/ResumoFinanceiro.svelte';
  import Configuracoes from './components/Configuracoes.svelte';
  import ExportarDados from './components/ExportarDados.svelte';
  import Categorias from './components/Categorias.svelte';
  import ProfileSelector from './components/ProfileSelector.svelte';
  import { onMount } from 'svelte';

  // Valores padrão que serão usados para novos perfis
  const defaultValues = {
    valorHora: 0,
    salarioDesejado: 0,
    horasPorDia: 8,
    diasPorSemana: 5,
    semanasFeriasAno: 4,
    jobs: [],
    categorias: [
      { id: 1, nome: 'Website', cor: 'blue' },
      { id: 2, nome: 'Mobile App', cor: 'green' },
      { id: 3, nome: 'Design', cor: 'purple' },
      { id: 4, nome: 'Consultoria', cor: 'orange' }
    ],
    configuracoes: {
      horasTrabalhoRegular: 8,
      diasTrabalhoRegular: 5,
      taxaUrgencia: 1.2,
      taxaComplexidade: 1.3,
      moeda: 'BRL',
      impostos: 0.06
    }
  };

  // Estado atual
  let valorHora = defaultValues.valorHora;
  let salarioDesejado = defaultValues.salarioDesejado;
  let horasPorDia = defaultValues.horasPorDia;
  let diasPorSemana = defaultValues.diasPorSemana;
  let semanasFeriasAno = defaultValues.semanasFeriasAno;
  let jobs = [...defaultValues.jobs];
  let categorias = [...defaultValues.categorias];
  let configuracoes = { ...defaultValues.configuracoes };

  let activeTab = 'Projetos';
  let selectedProfile = null;
  let showProfileSelector = true;

  function resetToDefaults() {
    valorHora = defaultValues.valorHora;
    salarioDesejado = defaultValues.salarioDesejado;
    horasPorDia = defaultValues.horasPorDia;
    diasPorSemana = defaultValues.diasPorSemana;
    semanasFeriasAno = defaultValues.semanasFeriasAno;
    jobs = [...defaultValues.jobs];
    categorias = [...defaultValues.categorias];
    configuracoes = { ...defaultValues.configuracoes };
  }

  function carregarDados() {
    if (typeof window !== 'undefined' && selectedProfile) {
      const dadosSalvos = localStorage.getItem(`jobscalc-dados-${selectedProfile.id}`);
      if (dadosSalvos) {
        const dados = JSON.parse(dadosSalvos);
        valorHora = dados.valorHora ?? defaultValues.valorHora;
        salarioDesejado = dados.salarioDesejado ?? defaultValues.salarioDesejado;
        horasPorDia = dados.horasPorDia ?? defaultValues.horasPorDia;
        diasPorSemana = dados.diasPorSemana ?? defaultValues.diasPorSemana;
        semanasFeriasAno = dados.semanasFeriasAno ?? defaultValues.semanasFeriasAno;
        jobs = dados.jobs ?? [...defaultValues.jobs];
        configuracoes = dados.configuracoes ?? { ...defaultValues.configuracoes };
        categorias = dados.categorias ?? [...defaultValues.categorias];
      } else {
        resetToDefaults();
      }
    }
  }

  function salvarDados() {
    if (typeof window !== 'undefined' && selectedProfile) {
      const dados = {
        valorHora,
        salarioDesejado,
        horasPorDia,
        diasPorSemana,
        semanasFeriasAno,
        jobs,
        configuracoes,
        categorias
      };
      localStorage.setItem(`jobscalc-dados-${selectedProfile.id}`, JSON.stringify(dados));
    }
  }

  // Não carrega dados no onMount, pois ainda não há perfil selecionado
  onMount(() => {
    // Apenas verifica se há um perfil salvo anteriormente
    const lastProfile = localStorage.getItem('jobscalc-last-profile');
    if (lastProfile) {
      try {
        selectedProfile = JSON.parse(lastProfile);
        showProfileSelector = false;
        carregarDados();
      } catch (error) {
        console.error('Erro ao carregar último perfil:', error);
      }
    }
  });

  // Salvar dados no localStorage quando houver mudanças
  $: if (typeof window !== 'undefined' && selectedProfile) {
    try {
      salvarDados();
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

  function handleProfileSelect(event) {
    selectedProfile = event.detail;
    showProfileSelector = false;
    // Salva o último perfil usado
    localStorage.setItem('jobscalc-last-profile', JSON.stringify(selectedProfile));
    // Carrega os dados do perfil
    carregarDados();
  }

  function handleLogout() {
    // Salva os dados atuais antes de trocar de perfil
    salvarDados();
    selectedProfile = null;
    showProfileSelector = true;
    // Reseta os dados para os valores padrão
    resetToDefaults();
  }

  function handleValoresAtualizados(event) {
    const valores = event.detail;
    salarioDesejado = valores.salarioDesejado;
    horasPorDia = valores.horasPorDia;
    diasPorSemana = valores.diasPorSemana;
    semanasFeriasAno = valores.semanasFeriasAno;
  }

  function handleValorHoraAtualizado(event) {
    valorHora = event.detail;
  }

  function handleConfiguracoesAtualizadas(event) {
    configuracoes = event.detail;
    salvarDados();
  }
</script>

{#if showProfileSelector}
  <ProfileSelector on:selectProfile={handleProfileSelect} />
{:else}
  <div class="min-h-screen bg-gray-900 text-gray-100">
    <!-- Header com informações do perfil -->
    <header class="bg-gray-800 border-b border-gray-700">
      <div class="max-w-7xl mx-auto flex justify-between items-center p-4">
        <div class="flex items-center space-x-3">
          {#if selectedProfile.avatar}
            <img
              src={selectedProfile.avatar}
              alt={selectedProfile.name}
              class="w-8 h-8 rounded-full"
            />
          {:else}
            <div class="w-8 h-8 rounded-full flex items-center justify-center text-sm text-white {selectedProfile.color}">
              {selectedProfile.initial}
            </div>
          {/if}
          <span class="text-white font-medium">{selectedProfile.name}</span>
        </div>

        <button
          class="text-gray-400 hover:text-white"
          on:click={handleLogout}
        >
          Trocar perfil
        </button>
      </div>
    </header>

    <!-- Conteúdo principal -->
    <main class="max-w-7xl mx-auto p-4">
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
            <ValorHora
              bind:valorHora={valorHora}
              salarioDesejado={salarioDesejado}
              horasPorDia={horasPorDia}
              diasPorSemana={diasPorSemana}
              semanasFeriasAno={semanasFeriasAno}
              on:valoresAtualizados={handleValoresAtualizados}
              on:valorHoraAtualizado={handleValorHoraAtualizado}
            />
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
          <Configuracoes bind:configuracoes on:configuracoesAtualizadas={handleConfiguracoesAtualizadas} />
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
{/if}
