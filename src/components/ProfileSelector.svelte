<script>
  import { onMount, createEventDispatcher } from 'svelte';
  const dispatch = createEventDispatcher();

  let profiles = [];
  let showAddProfile = false;
  let showDeleteConfirm = false;
  let showEditProfile = false;
  let profileToDelete = null;
  let profileToEdit = null;
  let newProfileName = '';
  let imageFile = null;
  let previewImage = null;

  onMount(() => {
    const savedProfiles = localStorage.getItem('jobscalc-profiles');
    if (savedProfiles) {
      profiles = JSON.parse(savedProfiles);
    }
  });

  function handleProfileSelect(profile) {
    dispatch('selectProfile', profile);
  }

  function handleImageChange(event) {
    const file = event.target.files[0];
    if (file) {
      const reader = new FileReader();
      reader.onload = (e) => {
        previewImage = e.target.result;
      };
      reader.readAsDataURL(file);
      imageFile = file;
    }
  }

  function addProfile() {
    const colors = [
      'bg-purple-500', 'bg-blue-500', 'bg-green-500', 
      'bg-yellow-500', 'bg-red-500', 'bg-indigo-500'
    ];
    
    const newProfile = {
      id: Date.now(),
      name: newProfileName,
      color: colors[Math.floor(Math.random() * colors.length)],
      initial: newProfileName.charAt(0).toUpperCase(),
      avatar: previewImage
    };

    profiles = [...profiles, newProfile];
    localStorage.setItem('jobscalc-profiles', JSON.stringify(profiles));
    newProfileName = '';
    previewImage = null;
    imageFile = null;
    showAddProfile = false;
  }

  function editProfile() {
    if (profileToEdit) {
      profiles = profiles.map(p => {
        if (p.id === profileToEdit.id) {
          return {
            ...p,
            avatar: previewImage || p.avatar
          };
        }
        return p;
      });
      localStorage.setItem('jobscalc-profiles', JSON.stringify(profiles));
      closeEditModal();
    }
  }

  function removeProfileImage() {
    previewImage = null;
    if (profileToEdit) {
      profiles = profiles.map(p => {
        if (p.id === profileToEdit.id) {
          return {
            ...p,
            avatar: null
          };
        }
        return p;
      });
      localStorage.setItem('jobscalc-profiles', JSON.stringify(profiles));
      closeEditModal();
    }
  }

  function openEditModal(profile, event) {
    event.stopPropagation();
    profileToEdit = profile;
    previewImage = profile.avatar;
    showEditProfile = true;
  }

  function closeEditModal() {
    showEditProfile = false;
    profileToEdit = null;
    previewImage = null;
    imageFile = null;
  }

  function confirmDelete(profile) {
    profileToDelete = profile;
    showDeleteConfirm = true;
  }

  function deleteProfile() {
    if (profileToDelete) {
      profiles = profiles.filter(p => p.id !== profileToDelete.id);
      localStorage.setItem('jobscalc-profiles', JSON.stringify(profiles));
      
      // Remove os dados associados ao perfil
      localStorage.removeItem(`jobscalc-dados-${profileToDelete.id}`);
      
      showDeleteConfirm = false;
      profileToDelete = null;
    }
  }

  function cancelDelete() {
    showDeleteConfirm = false;
    profileToDelete = null;
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
      <div class="group relative bg-gray-800 rounded-lg p-4 text-center hover:ring-2 hover:ring-white/20 transition-all">
        <!-- Menu de opções -->
        <div class="absolute top-2 right-2 opacity-0 group-hover:opacity-100 transition-opacity flex space-x-2">
          <button 
            class="text-gray-400 hover:text-indigo-400"
            on:click={(e) => openEditModal(profile, e)}
          >
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
              <path d="M13.586 3.586a2 2 0 112.828 2.828l-.793.793-2.828-2.828.793-.793zM11.379 5.793L3 14.172V17h2.828l8.38-8.379-2.83-2.828z" />
            </svg>
          </button>
          <button 
            class="text-gray-400 hover:text-red-500"
            on:click={(e) => {
              e.stopPropagation();
              confirmDelete(profile);
            }}
          >
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
              <path fill-rule="evenodd" d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z" clip-rule="evenodd" />
            </svg>
          </button>
        </div>

        <!-- Conteúdo do perfil -->
        <button
          class="w-full"
          on:click={() => handleProfileSelect(profile)}
        >
          <div class="flex flex-col items-center space-y-3">
            {#if profile.avatar}
              <img
                src={profile.avatar}
                alt={profile.name}
                class="w-20 h-20 rounded-full object-cover"
              />
            {:else}
              <div class="w-20 h-20 rounded-full flex items-center justify-center text-2xl text-white {profile.color}">
                {profile.initial}
              </div>
            {/if}
            <span class="text-white font-medium">{profile.name}</span>
          </div>
        </button>
      </div>
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
        
        <!-- Preview da imagem -->
        <div class="mb-4 flex justify-center">
          {#if previewImage}
            <img src={previewImage} alt="Preview" class="w-24 h-24 rounded-full object-cover" />
          {:else}
            <div class="w-24 h-24 rounded-full bg-gray-700 flex items-center justify-center">
              <svg xmlns="http://www.w3.org/2000/svg" class="h-12 w-12 text-gray-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z" />
              </svg>
            </div>
          {/if}
        </div>

        <!-- Input para upload de imagem -->
        <div class="mb-4">
          <label class="block text-sm font-medium text-gray-300 mb-2">
            Foto do Perfil (opcional)
          </label>
          <input
            type="file"
            accept="image/*"
            on:change={handleImageChange}
            class="block w-full text-sm text-gray-400
            file:mr-4 file:py-2 file:px-4
            file:rounded-full file:border-0
            file:text-sm file:font-semibold
            file:bg-indigo-500 file:text-white
            hover:file:bg-indigo-600
            cursor-pointer"
          />
        </div>

        <input
          type="text"
          bind:value={newProfileName}
          placeholder="Nome do perfil"
          class="w-full px-4 py-2 rounded-lg bg-gray-700 text-white placeholder-gray-400 mb-4"
        />
        
        <div class="flex justify-end space-x-3">
          <button
            class="px-4 py-2 text-gray-400 hover:text-white transition-colors"
            on:click={() => {
              showAddProfile = false;
              newProfileName = '';
              previewImage = null;
              imageFile = null;
            }}
          >
            Cancelar
          </button>
          <button
            class="px-4 py-2 bg-indigo-500 text-white rounded-lg hover:bg-indigo-600 transition-colors disabled:opacity-50 disabled:cursor-not-allowed"
            disabled={!newProfileName}
            on:click={addProfile}
          >
            Adicionar
          </button>
        </div>
      </div>
    </div>
  {/if}

  <!-- Modal de Edição de Perfil -->
  {#if showEditProfile}
    <div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4">
      <div class="bg-gray-800 rounded-lg p-6 w-full max-w-md">
        <h2 class="text-xl font-semibold text-white mb-4">Editar foto do perfil</h2>
        
        <!-- Preview da imagem -->
        <div class="mb-4 flex justify-center">
          {#if previewImage}
            <div class="relative">
              <img src={previewImage} alt="Preview" class="w-24 h-24 rounded-full object-cover" />
              <button
                class="absolute -top-2 -right-2 bg-red-500 text-white rounded-full p-1 hover:bg-red-600 transition-colors"
                on:click={() => removeProfileImage()}
              >
                <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" viewBox="0 0 20 20" fill="currentColor">
                  <path fill-rule="evenodd" d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z" clip-rule="evenodd" />
                </svg>
              </button>
            </div>
          {:else}
            <div class="w-24 h-24 rounded-full bg-gray-700 flex items-center justify-center">
              <svg xmlns="http://www.w3.org/2000/svg" class="h-12 w-12 text-gray-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z" />
              </svg>
            </div>
          {/if}
        </div>

        <!-- Input para upload de imagem -->
        <div class="mb-4">
          <label class="block text-sm font-medium text-gray-300 mb-2">
            Nova foto do perfil
          </label>
          <input
            type="file"
            accept="image/*"
            on:change={handleImageChange}
            class="block w-full text-sm text-gray-400
            file:mr-4 file:py-2 file:px-4
            file:rounded-full file:border-0
            file:text-sm file:font-semibold
            file:bg-indigo-500 file:text-white
            hover:file:bg-indigo-600
            cursor-pointer"
          />
        </div>
        
        <div class="flex justify-end space-x-3">
          <button
            class="px-4 py-2 text-gray-400 hover:text-white transition-colors"
            on:click={closeEditModal}
          >
            Cancelar
          </button>
          <button
            class="px-4 py-2 bg-indigo-500 text-white rounded-lg hover:bg-indigo-600 transition-colors"
            on:click={editProfile}
          >
            Salvar
          </button>
        </div>
      </div>
    </div>
  {/if}

  <!-- Modal de Confirmação de Exclusão -->
  {#if showDeleteConfirm}
    <div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4">
      <div class="bg-gray-800 rounded-lg p-6 w-full max-w-md">
        <h2 class="text-xl font-semibold text-white mb-2">Excluir perfil</h2>
        <p class="text-gray-400 mb-4">
          Tem certeza que deseja excluir o perfil "{profileToDelete?.name}"? Esta ação não pode ser desfeita.
        </p>
        
        <div class="flex justify-end space-x-3">
          <button
            class="px-4 py-2 text-gray-400 hover:text-white transition-colors"
            on:click={cancelDelete}
          >
            Cancelar
          </button>
          <button
            class="px-4 py-2 bg-red-500 text-white rounded-lg hover:bg-red-600 transition-colors"
            on:click={deleteProfile}
          >
            Excluir
          </button>
        </div>
      </div>
    </div>
  {/if}
</div>
