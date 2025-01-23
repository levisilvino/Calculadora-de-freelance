<script>
  export let jobs = [];
  export let configuracoes;

  function formatMoney(value) {
    return new Intl.NumberFormat('pt-BR', { 
      style: 'currency', 
      currency: 'BRL' 
    }).format(value);
  }

  $: totalHoras = jobs.reduce((total, job) => total + job.horasEstimadas, 0);
  $: totalValor = jobs.reduce((total, job) => total + job.valorTotal, 0);
  $: mediaPorProjeto = jobs.length > 0 ? totalValor / jobs.length : 0;
  
  // Calcular total de impostos
  $: totalImpostos = jobs.reduce((total, job) => {
    const valorBase = job.valorTotal / (1 + configuracoes.impostos);
    const imposto = job.valorTotal - valorBase;
    return total + imposto;
  }, 0);

  // Calcular valor sem impostos
  $: valorSemImpostos = totalValor - totalImpostos;

  // Status dos projetos
  $: projetosAFazer = jobs.filter(job => job.status === 'a_fazer').length;
  $: projetosFeitos = jobs.filter(job => job.status === 'feito').length;
  $: projetosPagos = jobs.filter(job => job.status === 'pago').length;
  $: projetosAtrasados = jobs.filter(job => job.status === 'atrasado').length;
</script>

<div class="space-y-6">
  <div class="bg-gray-800 rounded-lg shadow-md p-6">
    <h2 class="text-xl font-semibold mb-4 text-gray-200">Estatísticas</h2>
    
    <div class="space-y-4">
      <div class="flex justify-between items-center">
        <span class="text-sm text-gray-400">Total de Projetos</span>
        <span class="text-sm font-medium text-indigo-400">{jobs.length}</span>
      </div>
      
      <div class="flex justify-between items-center">
        <span class="text-sm text-gray-400">Total de Horas</span>
        <span class="text-sm font-medium text-indigo-400">{totalHoras}h</span>
      </div>
      
      <div class="flex justify-between items-center">
        <span class="text-sm text-gray-400">Média por Projeto</span>
        <span class="text-sm font-medium text-indigo-400">{formatMoney(mediaPorProjeto)}</span>
      </div>
    </div>
  </div>

  <div class="bg-gray-800 rounded-lg shadow-md p-6">
    <h3 class="text-lg font-semibold text-gray-200 mb-4">Status dos Projetos</h3>
    <div class="space-y-3">
      <div class="flex justify-between items-center">
        <span class="text-sm text-gray-400">A Fazer</span>
        <span class="text-sm font-medium text-yellow-400">{projetosAFazer} de {jobs.length}</span>
      </div>
      <div class="flex justify-between items-center">
        <span class="text-sm text-gray-400">Feitos</span>
        <span class="text-sm font-medium text-green-400">{projetosFeitos} de {jobs.length}</span>
      </div>
      <div class="flex justify-between items-center">
        <span class="text-sm text-gray-400">Pagos</span>
        <span class="text-sm font-medium text-indigo-400">{projetosPagos} de {jobs.length}</span>
      </div>
      <div class="flex justify-between items-center">
        <span class="text-sm text-gray-400">Atrasados</span>
        <span class="text-sm font-medium text-red-400">{projetosAtrasados} de {jobs.length}</span>
      </div>
    </div>
  </div>

  <div class="bg-gray-800 rounded-lg shadow-md p-6">
    <h3 class="text-lg font-semibold text-gray-200 mb-4">Valores</h3>
    <div class="space-y-3">
      <div class="flex justify-between items-center">
        <span class="text-sm text-gray-400">Valor sem Impostos</span>
        <span class="text-sm font-medium text-indigo-400">{formatMoney(valorSemImpostos)}</span>
      </div>
      <div class="flex justify-between items-center">
        <span class="text-sm text-gray-400">Total de Impostos</span>
        <span class="text-sm font-medium text-indigo-400">{formatMoney(totalImpostos)}</span>
      </div>
      <div class="flex justify-between items-center">
        <span class="text-sm text-gray-400">Valor Total dos Projetos</span>
        <span class="text-sm font-medium text-indigo-400">{formatMoney(totalValor)}</span>
      </div>
    </div>
  </div>
</div>
