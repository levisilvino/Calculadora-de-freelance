<script>
  export let configuracoes = {
    taxaUrgencia: 1.2,
    taxaComplexidade: 1.3,
    moeda: 'BRL',
    impostos: 0.06
  };

  let taxaUrgenciaPorcentagem = ((configuracoes.taxaUrgencia - 1) * 100) || 20;
  let taxaComplexidadePorcentagem = ((configuracoes.taxaComplexidade - 1) * 100) || 30;
  let impostoPorcentagem = (configuracoes.impostos * 100) || 6;
  let valorExemplo = 1000;

  // Estados para controlar a edição
  let editandoUrgencia = false;
  let editandoComplexidade = false;
  let editandoImpostos = false;

  // Função para focar o input quando ele aparecer
  function focusInput(node) {
    node.focus();
  }

  // Função para finalizar edição ao pressionar Enter ou perder foco
  function handleKeydown(event, tipo) {
    if (event.key === 'Enter' || event.key === 'Escape') {
      switch(tipo) {
        case 'urgencia': editandoUrgencia = false; break;
        case 'complexidade': editandoComplexidade = false; break;
        case 'impostos': editandoImpostos = false; break;
      }
    }
  }

  $: {
    // Limita as porcentagens a 2 casas decimais
    taxaUrgenciaPorcentagem = Number(taxaUrgenciaPorcentagem.toFixed(2));
    taxaComplexidadePorcentagem = Number(taxaComplexidadePorcentagem.toFixed(2));
    impostoPorcentagem = Number(impostoPorcentagem.toFixed(2));
    
    configuracoes.taxaUrgencia = 1 + (taxaUrgenciaPorcentagem / 100);
    configuracoes.taxaComplexidade = 1 + (taxaComplexidadePorcentagem / 100);
    configuracoes.impostos = impostoPorcentagem / 100;
  }

  const moedas = [
    { valor: 'BRL', nome: 'Real Brasileiro', simbolo: 'R$', local: 'pt-BR' },
    { valor: 'USD', nome: 'Dólar Americano', simbolo: '$', local: 'en-US' },
    { valor: 'EUR', nome: 'Euro', simbolo: '€', local: 'de-DE' }
  ];

  function formatMoney(value, currency = configuracoes.moeda) {
    const moeda = moedas.find(m => m.valor === currency);
    return new Intl.NumberFormat(moeda.local, { 
      style: 'currency', 
      currency: currency 
    }).format(value);
  }

  function calcularValorFinal(valor) {
    // 1. Calcula o valor base com taxas de urgência e complexidade
    const valorComTaxas = valor * configuracoes.taxaUrgencia * configuracoes.taxaComplexidade;
    // 2. Adiciona os impostos sobre o valor com taxas
    const valorComImpostos = valorComTaxas * (1 + configuracoes.impostos);
    return valorComImpostos;
  }

  function handleSave() {
    const el = document.createElement('div');
    el.className = 'fixed bottom-4 right-4 bg-green-500 text-white px-4 py-2 rounded-lg shadow-lg';
    el.textContent = 'Configurações salvas com sucesso!';
    document.body.appendChild(el);
    setTimeout(() => el.remove(), 3000);
  }
</script>

<div class="bg-gray-800 rounded-lg shadow-lg">
  <div class="p-6 border-b border-gray-700">
    <h2 class="text-2xl font-semibold text-gray-200">Configurações</h2>
    <p class="mt-1 text-gray-400">Personalize as configurações da sua calculadora</p>
  </div>

  <div class="p-6 space-y-8">
    <!-- Moeda -->
    <section>
      <div class="flex items-center justify-between mb-4">
        <div>
          <h3 class="text-lg font-medium text-gray-200">Moeda</h3>
          <p class="text-sm text-gray-400">Escolha a moeda para exibir os valores</p>
        </div>
        <span class="text-sm px-2 py-1 bg-gray-700 rounded text-gray-300">
          Atual: {moedas.find(m => m.valor === configuracoes.moeda).simbolo}
        </span>
      </div>
      
      <div class="grid grid-cols-1 sm:grid-cols-3 gap-3">
        {#each moedas as moeda}
          <button
            class="group relative p-4 rounded-lg border transition-all duration-200 {configuracoes.moeda === moeda.valor ? 'bg-indigo-500 border-indigo-400' : 'border-gray-600 hover:border-indigo-400'}"
            on:click={() => configuracoes.moeda = moeda.valor}
          >
            <div class="flex items-center justify-between">
              <div>
                <span class="block font-medium {configuracoes.moeda === moeda.valor ? 'text-white' : 'text-gray-300 group-hover:text-indigo-400'}">
                  {moeda.simbolo}
                </span>
                <span class="text-sm {configuracoes.moeda === moeda.valor ? 'text-indigo-200' : 'text-gray-400'}">
                  {moeda.nome}
                </span>
              </div>
              {#if configuracoes.moeda === moeda.valor}
                <svg class="w-5 h-5 text-white" viewBox="0 0 20 20" fill="currentColor">
                  <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd" />
                </svg>
              {/if}
            </div>
          </button>
        {/each}
      </div>
    </section>

    <!-- Taxas -->
    <section>
      <div class="mb-4">
        <h3 class="text-lg font-medium text-gray-200">Taxas e Impostos</h3>
        <p class="text-sm text-gray-400">Configure as taxas adicionais e impostos</p>
      </div>

      <div class="space-y-6">
        <!-- Taxa de Urgência -->
        <div class="bg-gray-700/30 rounded-lg p-4">
          <div class="flex items-center justify-between mb-2">
            <label class="text-sm font-medium text-gray-300">
              Taxa de Urgência
            </label>
            <div class="flex items-center gap-1">
              <span class="text-sm font-mono text-indigo-400">+</span>
              {#if editandoUrgencia}
                <input
                  type="number"
                  bind:value={taxaUrgenciaPorcentagem}
                  min="0"
                  max="100"
                  step="0.01"
                  use:focusInput
                  on:blur={() => editandoUrgencia = false}
                  on:keydown={(e) => handleKeydown(e, 'urgencia')}
                  class="w-16 text-sm font-mono bg-gray-700 px-2 py-1 rounded text-indigo-400 focus:ring-2 focus:ring-indigo-500 focus:outline-none"
                />
              {:else}
                <button
                  on:click={() => editandoUrgencia = true}
                  class="text-sm font-mono bg-gray-700 px-2 py-1 rounded text-indigo-400 hover:bg-gray-600 transition-colors"
                >
                  {taxaUrgenciaPorcentagem.toFixed(2)}
                </button>
              {/if}
              <span class="text-sm font-mono text-indigo-400">%</span>
            </div>
          </div>
          
          <div class="flex items-center space-x-4">
            <input
              type="range"
              bind:value={taxaUrgenciaPorcentagem}
              min="0"
              max="100"
              step="5"
              class="flex-1 h-2 bg-gray-600 rounded-lg appearance-none cursor-pointer accent-indigo-500"
            >
          </div>
          
          <p class="mt-2 text-sm text-gray-400">
            Valor adicional: {formatMoney(valorExemplo * (configuracoes.taxaUrgencia - 1))}
          </p>
        </div>

        <!-- Taxa de Complexidade -->
        <div class="bg-gray-700/30 rounded-lg p-4">
          <div class="flex items-center justify-between mb-2">
            <label class="text-sm font-medium text-gray-300">
              Taxa de Complexidade
            </label>
            <div class="flex items-center gap-1">
              <span class="text-sm font-mono text-indigo-400">+</span>
              {#if editandoComplexidade}
                <input
                  type="number"
                  bind:value={taxaComplexidadePorcentagem}
                  min="0"
                  max="100"
                  step="0.01"
                  use:focusInput
                  on:blur={() => editandoComplexidade = false}
                  on:keydown={(e) => handleKeydown(e, 'complexidade')}
                  class="w-16 text-sm font-mono bg-gray-700 px-2 py-1 rounded text-indigo-400 focus:ring-2 focus:ring-indigo-500 focus:outline-none"
                />
              {:else}
                <button
                  on:click={() => editandoComplexidade = true}
                  class="text-sm font-mono bg-gray-700 px-2 py-1 rounded text-indigo-400 hover:bg-gray-600 transition-colors"
                >
                  {taxaComplexidadePorcentagem.toFixed(2)}
                </button>
              {/if}
              <span class="text-sm font-mono text-indigo-400">%</span>
            </div>
          </div>
          
          <div class="flex items-center space-x-4">
            <input
              type="range"
              bind:value={taxaComplexidadePorcentagem}
              min="0"
              max="100"
              step="5"
              class="flex-1 h-2 bg-gray-600 rounded-lg appearance-none cursor-pointer accent-indigo-500"
            >
          </div>
          
          <p class="mt-2 text-sm text-gray-400">
            Valor adicional: {formatMoney(valorExemplo * (configuracoes.taxaComplexidade - 1))}
          </p>
        </div>

        <!-- Impostos -->
        <div class="bg-gray-700/30 rounded-lg p-4">
          <div class="flex items-center justify-between mb-2">
            <div>
              <label class="text-sm font-medium text-gray-300">
                Impostos
              </label>
              <p class="text-xs text-gray-400 mt-0.5">
                Impostos são calculados sobre o valor final após taxas adicionais
              </p>
            </div>
            <div class="flex items-center gap-1">
              {#if editandoImpostos}
                <input
                  type="number"
                  bind:value={impostoPorcentagem}
                  min="0"
                  max="40"
                  step="0.01"
                  use:focusInput
                  on:blur={() => editandoImpostos = false}
                  on:keydown={(e) => handleKeydown(e, 'impostos')}
                  class="w-16 text-sm font-mono bg-gray-700 px-2 py-1 rounded text-yellow-400 focus:ring-2 focus:ring-yellow-500 focus:outline-none"
                />
              {:else}
                <button
                  on:click={() => editandoImpostos = true}
                  class="text-sm font-mono bg-gray-700 px-2 py-1 rounded text-yellow-400 hover:bg-gray-600 transition-colors"
                >
                  {impostoPorcentagem.toFixed(2)}
                </button>
              {/if}
              <span class="text-sm font-mono text-yellow-400">%</span>
            </div>
          </div>
          
          <div class="flex items-center space-x-4">
            <input
              type="range"
              bind:value={impostoPorcentagem}
              min="0"
              max="40"
              step="0.5"
              class="flex-1 h-2 bg-gray-600 rounded-lg appearance-none cursor-pointer accent-yellow-500"
            >
          </div>
          
          <p class="mt-2 text-sm text-gray-400">
            Valor dos impostos: {formatMoney(valorExemplo * configuracoes.taxaUrgencia * configuracoes.taxaComplexidade * configuracoes.impostos)}
          </p>
        </div>
      </div>
    </section>

    <!-- Exemplo -->
    <section class="bg-gradient-to-br from-gray-700/30 to-gray-700/10 rounded-lg p-6">
      <div class="mb-4">
        <h3 class="text-lg font-medium text-gray-200">Simulação de Projeto</h3>
        <p class="text-sm text-gray-400">Veja como as taxas e impostos afetam o valor final</p>
      </div>

      <div class="space-y-4">
        <div>
          <label class="block text-sm font-medium text-gray-300 mb-2">
            Valor Base do Projeto
          </label>
          <div class="relative">
            <input
              type="number"
              bind:value={valorExemplo}
              min="0"
              step="100"
              class="w-full px-3 py-2 bg-gray-700 border border-gray-600 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 text-gray-100 pr-16"
            >
            <div class="absolute inset-y-0 right-0 pr-3 flex items-center pointer-events-none">
              <span class="text-gray-400">{configuracoes.moeda}</span>
            </div>
          </div>
        </div>

        <div class="bg-gray-700/30 rounded-lg p-4">
          <ul class="space-y-2 text-sm">
            <li class="flex justify-between">
              <span class="text-gray-400">Valor Base:</span>
              <span class="text-gray-200">{formatMoney(valorExemplo)}</span>
            </li>
            {#if taxaUrgenciaPorcentagem > 0}
              <li class="flex justify-between">
                <span class="text-gray-400">+ Taxa de Urgência ({taxaUrgenciaPorcentagem.toFixed(2)}%):</span>
                <span class="text-gray-200">{formatMoney(valorExemplo * (configuracoes.taxaUrgencia - 1))}</span>
              </li>
            {/if}
            {#if taxaComplexidadePorcentagem > 0}
              <li class="flex justify-between">
                <span class="text-gray-400">+ Taxa de Complexidade ({taxaComplexidadePorcentagem.toFixed(2)}%):</span>
                <span class="text-gray-200">{formatMoney(valorExemplo * (configuracoes.taxaComplexidade - 1))}</span>
              </li>
            {/if}
            <li class="flex justify-between border-t border-gray-600 pt-2">
              <span class="text-gray-400">Subtotal (com taxas):</span>
              <span class="text-gray-200">{formatMoney(valorExemplo * configuracoes.taxaUrgencia * configuracoes.taxaComplexidade)}</span>
            </li>
            {#if impostoPorcentagem > 0}
              <li class="flex justify-between">
                <span class="text-gray-400">+ Impostos ({impostoPorcentagem.toFixed(2)}%):</span>
                <span class="text-yellow-400">{formatMoney(valorExemplo * configuracoes.taxaUrgencia * configuracoes.taxaComplexidade * configuracoes.impostos)}</span>
              </li>
            {/if}
            <li class="flex justify-between pt-2 border-t border-gray-600">
              <span class="font-medium text-gray-200">Valor Final:</span>
              <span class="font-medium text-indigo-400">
                {formatMoney(calcularValorFinal(valorExemplo))}
              </span>
            </li>
          </ul>
        </div>

        <div class="bg-gray-800/50 rounded-lg p-4 text-sm">
          <h4 class="font-medium text-gray-300 mb-2">Como é calculado?</h4>
          <ol class="list-decimal list-inside space-y-1 text-gray-400">
            <li>Valor base: o preço inicial do projeto</li>
            <li>Aplicação das taxas adicionais (urgência e complexidade)</li>
            <li>Cálculo dos impostos sobre o valor com taxas</li>
          </ol>
        </div>
      </div>
    </section>
  </div>

  <div class="p-6 bg-gray-700/30 border-t border-gray-700">
    <button
      on:click={handleSave}
      class="w-full px-4 py-3 bg-gradient-to-r from-indigo-500 to-indigo-600 text-white rounded-lg font-medium hover:from-indigo-600 hover:to-indigo-700 transition-colors flex items-center justify-center gap-2"
    >
      <svg class="w-5 h-5" viewBox="0 0 20 20" fill="currentColor">
        <path fill-rule="evenodd" d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z" clip-rule="evenodd" />
      </svg>
      Salvar Configurações
    </button>
  </div>
</div>
