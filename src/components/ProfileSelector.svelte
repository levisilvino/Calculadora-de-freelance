`<script>
  import { onMount, createEventDispatcher } from 'svelte';
  const dispatch = createEventDispatcher();

  let profiles = [];
  let showAddProfile = false;
  let newProfileName = '';

  onMount(() => {
    const savedProfiles = localStorage.getItem('jobscalc-profiles');
    if (savedProfiles) {
      profiles = JSON.parse(savedProfiles);
    }
  });

  function handleProfileSelect(profile) {
    dispatch('selectProfile', profile);
  }

  function addProfile() {
    const colors = [
      'bg-purple-500', 'bg-blue-500', 'bg-green-500', 
      'bg-yellow-500', 'bg-red-500', 'bg-indigo-500'
    ];
    
    const newProfile = {
      id: Date.now(), // Usando timestamp como ID único
      name: newProfileName,
      color: colors[Math.floor(Math.random() * colors.length)],
      initial: newProfileName.charAt(0).toUpperCase(),
      avatar: null
    };

    profiles = [...profiles, newProfile];
    localStorage.setItem('jobscalc-profiles', JSON.stringify(profiles));
    newProfileName = '';
    showAddProfile = false;
  }
</script>

<div class="min-h-screen bg-gray-900 flex flex-col items-center justify-center p-4">
  <div class="text-center mb-8">
    <h1 class="text-2xl font-semibold text-white mb-2">Quem está usando a Calculadora?</h1>
    <p class="text-gray-400">
      Crie perfis diferentes para separar seus cálculos e configurações
    </p>
  </div>

  <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4 max-w-4xl">
    {#each profiles as profile (profile.id)}
      <button
        class="group relative bg-gray-800 rounded-lg p-4 text-center hover:ring-2 hover:ring-white/20 transition-all"
        on:click={() => handleProfileSelect(profile)}
      >
        <div class="absolute top-2 right-2 opacity-0 group-hover:opacity-100 transition-opacity">
          <button class="text-gray-400 hover:text-white">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
              <path d="M6 10a2 2 0 11-4 0 2 2 0 014 0zM12 10a2 2 0 11-4 0 2 2 0 014 0zM16 12a2 2 0 100-4 2 2 0 000 4z" />
            </svg>
          </button>
        </div>

        <div class="flex flex-col items-center space-y-3">
          {#if profile.avatar}
            <img
              src={profile.avatar}
              alt={profile.name}
              class="w-20 h-20 rounded-full"
            />
          {:else}
            <div class="w-20 h-20 rounded-full flex items-center justify-center text-2xl text-white {profile.color}">
              {profile.initial}
            </div>
          {/if}
          <span class="text-white font-medium">{profile.name}</span>
        </div>
      </button>
    {/each}

    <!-- Botão Adicionar -->
    <button
      class="bg-gray-800 rounded-lg p-4 text-center hover:ring-2 hover:ring-white/20 transition-all"
      on:click={() => showAddProfile = true}
    >
      <div class="flex flex-col items-center space-y-3">
        <div class="w-20 h-20 rounded-full flex items-center justify-center bg-gray-700 text-white">
          <svg xmlns="http://www.w3.org/2000/svg" class="h-10 w-10" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4" />
          </svg>
        </div>
        <span class="text-white font-medium">Adicionar</span>
      </div>
    </button>
  </div>

  <!-- Modal Adicionar Perfil -->
  {#if showAddProfile}
    <div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4">
      <div class="bg-gray-800 rounded-lg p-6 w-full max-w-md">
        <h2 class="text-xl font-semibold text-white mb-4">Adicionar novo perfil</h2>
        <input
          type="text"
          bind:value={newProfileName}
          placeholder="Nome do perfil"
          class="w-full p-2 rounded bg-gray-700 text-white border border-gray-600 focus:border-blue-500 focus:ring-1 focus:ring-blue-500 mb-4"
        />
        <div class="flex justify-end space-x-3">
          <button
            class="px-4 py-2 text-gray-400 hover:text-white"
            on:click={() => showAddProfile = false}
          >
            Cancelar
          </button>
          <button
            class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600"
            on:click={addProfile}
            disabled={!newProfileName}
          >
            Adicionar
          </button>
        </div>
      </div>
    </div>
  {/if}

  <!-- Modo Visitante -->
  <div class="mt-8">
    <button class="flex items-center space-x-2 text-blue-400 hover:text-blue-300">
      <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
        <path d="M8 9a3 3 0 100-6 3 3 0 000 6zM8 11a6 6 0 016 6H2a6 6 0 016-6zM16 7a1 1 0 10-2 0v1h-1a1 1 0 100 2h1v1a1 1 0 102 0v-1h1a1 1 0 100-2h-1V7z" />
      </svg>
      <span>Modo visitante</span>
    </button>
  </div>
</div>
`
