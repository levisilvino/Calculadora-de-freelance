<script>
  import { createEventDispatcher } from 'svelte';
  const dispatch = createEventDispatcher();

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
    
    const novasConfiguracoes = {
      ...configuracoes,
      taxaUrgencia: 1 + (taxaUrgenciaPorcentagem / 100),
      taxaComplexidade: 1 + (taxaComplexidadePorcentagem / 100),
      impostos: impostoPorcentagem / 100
    };

    if (JSON.stringify(novasConfiguracoes) !== JSON.stringify(configuracoes)) {
      configuracoes = novasConfiguracoes;
      dispatch('configuracoesAtualizadas', configuracoes);
    }
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
    dispatch('configuracoesAtualizadas', configuracoes);
    
    // Mostra mensagem de sucesso
    const el = document.createElement('div');
    el.className = 'fixed bottom-4 right-4 bg-green-500 text-white px-4 py-2 rounded-lg shadow-lg';
    el.textContent = 'Configurações salvas com sucesso!';
    document.body.appendChild(el);
    setTimeout(() => el.remove(), 3000);
  }

  function handleMoedaChange(event) {
    configuracoes.moeda = event.target.value;
    dispatch('configuracoesAtualizadas', configuracoes);
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
          <p class="text-sm text-gray-400">Selecione a moeda para seus cálculos</p>
        </div>
      </div>

      <select
        bind:value={configuracoes.moeda}
        on:change={handleMoedaChange}
        class="w-full px-3 py-2 bg-gray-700/30 border border-gray-600 rounded-lg text-gray-200 focus:ring-1 focus:ring-indigo-500 focus:border-indigo-500"
      >
        {#each moedas as moeda}
          <option value={moeda.valor}>
            {moeda.nome} ({moeda.simbolo})
          </option>
        {/each}
      </select>
    </section>

    <!-- Taxas e Impostos -->
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
            {#if !editandoUrgencia}
              <button
                class="text-sm text-indigo-400 hover:text-indigo-300"
                on:click={() => editandoUrgencia = true}
              >
                Editar
              </button>
            {/if}
          </div>
          
          {#if editandoUrgencia}
            <div class="flex items-center">
              <input
                type="number"
                bind:value={taxaUrgenciaPorcentagem}
                on:keydown={e => handleKeydown(e, 'urgencia')}
                on:blur={() => editandoUrgencia = false}
                use:focusInput
                min="0"
                max="1000"
                step="0.1"
                class="w-20 px-2 py-1 bg-gray-600 border border-gray-500 rounded text-white text-right"
              />
              <span class="ml-2 text-gray-400">%</span>
            </div>
          {:else}
            <p class="text-2xl font-medium text-gray-200">
              {taxaUrgenciaPorcentagem}%
            </p>
          {/if}
          
          <p class="mt-1 text-sm text-gray-400">
            Taxa adicional para projetos urgentes
          </p>
        </div>

        <!-- Taxa de Complexidade -->
        <div class="bg-gray-700/30 rounded-lg p-4">
          <div class="flex items-center justify-between mb-2">
            <label class="text-sm font-medium text-gray-300">
              Taxa de Complexidade
            </label>
            {#if !editandoComplexidade}
              <button
                class="text-sm text-indigo-400 hover:text-indigo-300"
                on:click={() => editandoComplexidade = true}
              >
                Editar
              </button>
            {/if}
          </div>
          
          {#if editandoComplexidade}
            <div class="flex items-center">
              <input
                type="number"
                bind:value={taxaComplexidadePorcentagem}
                on:keydown={e => handleKeydown(e, 'complexidade')}
                on:blur={() => editandoComplexidade = false}
                use:focusInput
                min="0"
                max="1000"
                step="0.1"
                class="w-20 px-2 py-1 bg-gray-600 border border-gray-500 rounded text-white text-right"
              />
              <span class="ml-2 text-gray-400">%</span>
            </div>
          {:else}
            <p class="text-2xl font-medium text-gray-200">
              {taxaComplexidadePorcentagem}%
            </p>
          {/if}
          
          <p class="mt-1 text-sm text-gray-400">
            Taxa adicional para projetos complexos
          </p>
        </div>

        <!-- Impostos -->
        <div class="bg-gray-700/30 rounded-lg p-4">
          <div class="flex items-center justify-between mb-2">
            <div>
              <label class="text-sm font-medium text-gray-300">
                Impostos
              </label>
              {#if !editandoImpostos}
                <button
                  class="ml-4 text-sm text-indigo-400 hover:text-indigo-300"
                  on:click={() => editandoImpostos = true}
                >
                  Editar
                </button>
              {/if}
            </div>
          </div>
          
          {#if editandoImpostos}
            <div class="flex items-center">
              <input
                type="number"
                bind:value={impostoPorcentagem}
                on:keydown={e => handleKeydown(e, 'impostos')}
                on:blur={() => editandoImpostos = false}
                use:focusInput
                min="0"
                max="100"
                step="0.1"
                class="w-20 px-2 py-1 bg-gray-600 border border-gray-500 rounded text-white text-right"
              />
              <span class="ml-2 text-gray-400">%</span>
            </div>
          {:else}
            <p class="text-2xl font-medium text-gray-200">
              {impostoPorcentagem}%
            </p>
          {/if}
          
          <p class="mt-1 text-sm text-gray-400">
            Impostos sobre o valor final
          </p>
        </div>
      </div>
    </section>

    <!-- Exemplo -->
    <section class="bg-gradient-to-br from-gray-700/30 to-gray-700/10 rounded-lg p-6">
      <div class="mb-4">
        <h3 class="text-lg font-medium text-gray-200">Exemplo de Cálculo</h3>
        <p class="text-sm text-gray-400">Veja como as taxas afetam o valor final</p>
      </div>

      <div class="space-y-4">
        <div class="flex items-center space-x-4">
          <label class="text-sm text-gray-400">Valor Base:</label>
          <input
            type="number"
            bind:value={valorExemplo}
            class="w-32 px-3 py-1.5 bg-gray-700/50 border border-gray-600 rounded text-white text-right"
          />
        </div>

        <div class="space-y-2">
          <div class="flex justify-between text-sm">
            <span class="text-gray-400">Valor Base</span>
            <span class="text-gray-200">{formatMoney(valorExemplo)}</span>
          </div>
          
          <div class="flex justify-between text-sm">
            <span class="text-gray-400">+ Taxa de Urgência ({taxaUrgenciaPorcentagem}%)</span>
            <span class="text-gray-200">{formatMoney(valorExemplo * (configuracoes.taxaUrgencia - 1))}</span>
          </div>
          
          <div class="flex justify-between text-sm">
            <span class="text-gray-400">+ Taxa de Complexidade ({taxaComplexidadePorcentagem}%)</span>
            <span class="text-gray-200">{formatMoney(valorExemplo * (configuracoes.taxaComplexidade - 1))}</span>
          </div>
          
          <div class="flex justify-between text-sm">
            <span class="text-gray-400">+ Impostos ({impostoPorcentagem}%)</span>
            <span class="text-gray-200">{formatMoney(calcularValorFinal(valorExemplo) - (valorExemplo * configuracoes.taxaUrgencia * configuracoes.taxaComplexidade))}</span>
          </div>

          <div class="pt-2 mt-2 border-t border-gray-600">
            <div class="flex justify-between">
              <span class="font-medium text-gray-300">Valor Final</span>
              <span class="font-medium text-indigo-400">{formatMoney(calcularValorFinal(valorExemplo))}</span>
            </div>
          </div>
        </div>
      </div>
    </section>
  </div>

  <div class="p-6 bg-gray-700/30 border-t border-gray-700">
    <button
      on:click={handleSave}
      class="w-full px-4 py-2 bg-indigo-500 hover:bg-indigo-600 text-white font-medium rounded-lg transition-colors"
    >
      Salvar Configurações
    </button>
  </div>
</div>
