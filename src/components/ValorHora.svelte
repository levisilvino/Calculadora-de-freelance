<script>
  import { onMount } from 'svelte';
  import { fade } from 'svelte/transition';

  export let valorHora = 0;
  let salarioDesejado = 0;
  let horasPorDia = 8;
  let diasPorSemana = 5;
  let semanasFeriasAno = 4;
  let showCalculo = false;

  // Validar e ajustar valores
  function validarValores() {
    // Limitar horas por dia (máximo 24)
    horasPorDia = Math.min(Math.max(0, horasPorDia), 24);
    
    // Limitar dias por semana (máximo 7)
    diasPorSemana = Math.min(Math.max(0, diasPorSemana), 7);
    
    // Limitar semanas de férias (máximo 52)
    semanasFeriasAno = Math.min(Math.max(0, semanasFeriasAno), 52);
    
    // Garantir que salário não seja negativo
    salarioDesejado = Math.max(0, salarioDesejado);
  }

  onMount(() => {
    // Carregar valores do localStorage
    const savedSalario = localStorage.getItem('salarioDesejado');
    const savedHoras = localStorage.getItem('horasPorDia');
    const savedDias = localStorage.getItem('diasPorSemana');
    const savedFerias = localStorage.getItem('semanasFeriasAno');
    const savedValorHora = localStorage.getItem('valorHora');

    // Definir valores padrão apenas se não houver valores salvos
    salarioDesejado = savedSalario ? Number(savedSalario) : 0;
    horasPorDia = savedHoras ? Number(savedHoras) : 8;
    diasPorSemana = savedDias ? Number(savedDias) : 5;
    semanasFeriasAno = savedFerias ? Number(savedFerias) : 4;
    valorHora = savedValorHora ? Number(savedValorHora) : 0;

    console.log('Valores carregados:', {
      salarioDesejado,
      horasPorDia,
      diasPorSemana,
      semanasFeriasAno,
      valorHora
    });

    validarValores();
    
    // Adiciona um pequeno delay antes de calcular
    setTimeout(() => {
      if (salarioDesejado > 0) {
        calcularValorHora();
        console.log('Valor hora recalculado após delay:', valorHora);
      }
    }, 100);
  });

  function calcularValorHora() {
    validarValores();
    
    const salario = Number(salarioDesejado) || 0;
    const semanasTrabalhadasAno = 52 - Number(semanasFeriasAno);
    const horasTrabalhadasAno = Number(horasPorDia) * Number(diasPorSemana) * semanasTrabalhadasAno;
    const horasTrabalhadasMes = horasTrabalhadasAno / 12;
    
    if (horasTrabalhadasMes > 0 && salario > 0) {
      const valor = salario / horasTrabalhadasMes;
      const novoValorHora = Number(valor.toFixed(2));
      
      if (!isNaN(novoValorHora) && isFinite(novoValorHora) && novoValorHora > 0) {
        valorHora = novoValorHora;
        localStorage.setItem('valorHora', valorHora.toString());
        console.log('Cálculo:', {
          salario,
          horasTrabalhadasMes,
          valorHora
        });
      } else {
        valorHora = 0;
        localStorage.setItem('valorHora', '0');
      }
    } else {
      valorHora = 0;
      localStorage.setItem('valorHora', '0');
    }
  }

  function formatMoney(value) {
    return new Intl.NumberFormat('pt-BR', { 
      style: 'currency', 
      currency: 'BRL' 
    }).format(value);
  }

  // Valores calculados reativamente
  $: horasMensais = ((52 - semanasFeriasAno) * horasPorDia * diasPorSemana / 12).toFixed(1);
  $: valorHoraFormatado = formatMoney(valorHora);
  $: salarioFormatado = formatMoney(salarioDesejado);
  
  // Recalcular quando qualquer valor mudar
  $: {
    if (salarioDesejado || horasPorDia || diasPorSemana || semanasFeriasAno) {
      validarValores();
      calcularValorHora();
    }
  }

  function handleSalarioChange(e) {
    e.preventDefault();
    const valor = Number(e.target.value);
    salarioDesejado = valor;
    // Salvar imediatamente quando o usuário digita
    localStorage.setItem('salarioDesejado', valor.toString());
    // Recalcular valor hora
    calcularValorHora();
  }

  function handleHorasChange(e) {
    const valor = Number(e.target.value);
    horasPorDia = Math.min(Math.max(0, valor), 24);
    localStorage.setItem('horasPorDia', horasPorDia.toString());
    calcularValorHora();
  }

  function handleDiasChange(e) {
    const valor = Number(e.target.value);
    diasPorSemana = Math.min(Math.max(0, valor), 7);
    localStorage.setItem('diasPorSemana', diasPorSemana.toString());
    calcularValorHora();
  }

  function handleFeriasChange(e) {
    const valor = Number(e.target.value);
    semanasFeriasAno = Math.min(Math.max(0, valor), 52);
    localStorage.setItem('semanasFeriasAno', semanasFeriasAno.toString());
    calcularValorHora();
  }
</script>

<div class="max-w-sm mx-auto bg-gray-800 rounded-xl border border-gray-700 shadow-lg">
  <div class="p-5 space-y-4">
    <!-- Salário -->
    <div>
      <label for="salario" class="block text-sm text-gray-300 mb-1.5">
        Salário Mensal Desejado
      </label>
      <div class="relative">
        <div class="absolute inset-y-0 left-0 pl-2.5 flex items-center pointer-events-none">
          <span class="text-gray-400 text-sm">R$</span>
        </div>
        <input
          id="salario"
          type="number"
          bind:value={salarioDesejado}
          on:input={handleSalarioChange}
          on:keydown={(e) => {
            if (e.key === 'Enter') {
              e.preventDefault();
            }
          }}
          min="0"
          step="100"
          class="w-full pl-8 pr-3 py-1.5 bg-gray-700/50 border border-gray-600 rounded-lg
                 text-gray-100 text-sm
                 focus:ring-1 focus:ring-indigo-500 focus:border-indigo-500"
        />
      </div>
    </div>

    <!-- Configurações -->
    <div>
      <div class="text-sm text-gray-300 mb-2">Configurações de Tempo</div>
      <div class="grid grid-cols-3 gap-2">
        <div>
          <label for="horasDia" class="block text-xs text-gray-400 mb-1">
            Horas por Dia
          </label>
          <div class="relative">
            <input
              id="horasDia"
              type="number"
              bind:value={horasPorDia}
              on:input={handleHorasChange}
              min="1"
              max="24"
              class="w-full pl-2 pr-12 py-1.5 bg-gray-700/50 border border-gray-600 rounded-lg
                     text-gray-100 text-sm
                     focus:ring-1 focus:ring-indigo-500 focus:border-indigo-500"
            />
            <div class="absolute inset-y-0 right-0 pr-2 flex items-center pointer-events-none">
              <span class="text-gray-500 text-xs">horas</span>
            </div>
          </div>
        </div>

        <div>
          <label for="diasSemana" class="block text-xs text-gray-400 mb-1">
            Dias por Semana
          </label>
          <div class="relative">
            <input
              id="diasSemana"
              type="number"
              bind:value={diasPorSemana}
              on:input={handleDiasChange}
              min="1"
              max="7"
              class="w-full pl-2 pr-12 py-1.5 bg-gray-700/50 border border-gray-600 rounded-lg
                     text-gray-100 text-sm
                     focus:ring-1 focus:ring-indigo-500 focus:border-indigo-500"
            />
            <div class="absolute inset-y-0 right-0 pr-2 flex items-center pointer-events-none">
              <span class="text-gray-500 text-xs">dias</span>
            </div>
          </div>
        </div>

        <div>
          <label for="semanasFeriasAno" class="block text-xs text-gray-400 mb-1">
            Férias (semanas/ano)
          </label>
          <div class="relative">
            <input
              id="semanasFeriasAno"
              type="number"
              bind:value={semanasFeriasAno}
              on:input={handleFeriasChange}
              min="0"
              max="52"
              class="w-full pl-2 pr-12 py-1.5 bg-gray-700/50 border border-gray-600 rounded-lg
                     text-gray-100 text-sm
                     focus:ring-1 focus:ring-indigo-500 focus:border-indigo-500"
            />
            <div class="absolute inset-y-0 right-0 pr-2 flex items-center pointer-events-none">
              <span class="text-gray-500 text-xs">sem</span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Resultados -->
    <div class="grid grid-cols-2 gap-2 border-t border-gray-700 pt-4 mt-4">
      <div>
        <div class="text-xs text-gray-400">Horas Mensais</div>
        <div class="text-lg text-gray-100 font-medium">{((52 - semanasFeriasAno) * horasPorDia * diasPorSemana / 12).toFixed(1)}h</div>
      </div>
      <div>
        <div class="text-xs text-gray-400">Valor por Hora</div>
        <div class="text-lg text-indigo-400 font-medium">{formatMoney(valorHora)}</div>
      </div>
      <div class="col-span-2">
        <div class="text-xs text-gray-400">Salário Mensal</div>
        <div class="text-lg text-gray-100 font-medium">{formatMoney(salarioDesejado)}</div>
      </div>
    </div>

    <!-- Como é calculado -->
    <button
      class="w-full flex items-center gap-1.5 text-sm text-gray-400 hover:text-gray-300 transition-colors"
      on:click={() => showCalculo = !showCalculo}
    >
      <svg 
        class="w-4 h-4 transition-transform duration-200" 
        class:rotate-180={showCalculo}
        viewBox="0 0 20 20" 
        fill="currentColor"
      >
        <path fill-rule="evenodd" d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z" clip-rule="evenodd" />
      </svg>
      Como é calculado?
    </button>
    
    {#if showCalculo}
      <div class="text-xs text-gray-400 space-y-1 pt-1" transition:fade>
        <p>1. Total de semanas: 52 - {semanasFeriasAno} semanas de férias = {52 - semanasFeriasAno} semanas</p>
        <p>2. Horas por semana: {horasPorDia} horas × {diasPorSemana} dias = {horasPorDia * diasPorSemana} horas</p>
        <p>3. Média mensal: {((52 - semanasFeriasAno) * horasPorDia * diasPorSemana / 12).toFixed(1)} horas</p>
        <p>4. Valor por hora: {formatMoney(salarioDesejado)} ÷ {((52 - semanasFeriasAno) * horasPorDia * diasPorSemana / 12).toFixed(1)} horas = {formatMoney(valorHora)}/hora</p>
      </div>
    {/if}
  </div>
</div>

<style>
  input[type="number"]::-webkit-inner-spin-button,
  input[type="number"]::-webkit-outer-spin-button {
    -webkit-appearance: none;
    margin: 0;
  }
  input[type="number"] {
    -moz-appearance: textfield;
    appearance: textfield;
  }
</style>
