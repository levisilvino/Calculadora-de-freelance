<script>
  export let categorias = [
    { id: 1, nome: 'Website', cor: 'indigo' },
    { id: 2, nome: 'Mobile App', cor: 'emerald' },
    { id: 3, nome: 'Design', cor: 'amber' },
    { id: 4, nome: 'Consultoria', cor: 'rose' }
  ];

  let novaCategoria = { nome: '', cor: 'indigo' };
  let categoriaEditando = null;
  let showModal = false;
  let modalAction = '';

  const cores = [
    { valor: 'indigo', nome: 'Índigo' },
    { valor: 'emerald', nome: 'Esmeralda' },
    { valor: 'amber', nome: 'Âmbar' },
    { valor: 'rose', nome: 'Rosa' },
    { valor: 'violet', nome: 'Violeta' },
    { valor: 'cyan', nome: 'Ciano' }
  ];

  function handleSubmit() {
    if (novaCategoria.nome.trim()) {
      categorias = [...categorias, {
        id: Date.now(),
        nome: novaCategoria.nome,
        cor: novaCategoria.cor
      }];
      novaCategoria = { nome: '', cor: 'indigo' };
    }
  }

  function handleEdit(categoria) {
    categoriaEditando = { ...categoria };
    modalAction = 'edit';
    showModal = true;
  }

  function handleDelete(categoria) {
    categoriaEditando = categoria;
    modalAction = 'delete';
    showModal = true;
  }

  function confirmAction() {
    if (modalAction === 'edit') {
      categorias = categorias.map(cat => cat.id === categoriaEditando.id ? categoriaEditando : cat);
    } else if (modalAction === 'delete') {
      categorias = categorias.filter(cat => cat.id !== categoriaEditando.id);
    }
    closeModal();
  }

  function closeModal() {
    showModal = false;
    modalAction = '';
    categoriaEditando = null;
  }

  function getCorClasse(cor) {
    const classes = {
      indigo: 'bg-indigo-500/20 text-indigo-400',
      emerald: 'bg-emerald-500/20 text-emerald-400',
      amber: 'bg-amber-500/20 text-amber-400',
      rose: 'bg-rose-500/20 text-rose-400',
      violet: 'bg-violet-500/20 text-violet-400',
      cyan: 'bg-cyan-500/20 text-cyan-400'
    };
    return classes[cor] || classes.indigo;
  }
</script>

<div class="bg-gray-900 text-gray-100">
  <div class="space-y-6 p-6">
    <div class="flex items-center justify-between">
      <div class="flex-1">
        <div class="text-2xl font-bold text-transparent bg-clip-text bg-gradient-to-r from-indigo-400 to-indigo-600">
          Categorias
        </div>
        <div class="text-sm text-gray-400 mt-1">
          Organize seus projetos por categorias
        </div>
      </div>
      <div class="w-12 h-12 bg-gradient-to-br from-indigo-500 to-indigo-600 rounded-2xl flex items-center justify-center shadow-lg">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-white" viewBox="0 0 20 20" fill="currentColor">
          <path fill-rule="evenodd" d="M17.707 9.293a1 1 0 010 1.414l-7 7a1 1 0 01-1.414 0l-7-7A.997.997 0 012 10V5a3 3 0 013-3h5c.256 0 .512.098.707.293l7 7zM5 6a1 1 0 100-2 1 1 0 000 2z" clip-rule="evenodd" />
        </svg>
      </div>
    </div>

    <form on:submit|preventDefault={handleSubmit} class="bg-gray-800/50 rounded-xl p-4 border border-gray-600">
      <div class="flex gap-4">
        <div class="flex-1">
          <label for="nome-categoria" class="sr-only">Nome da Categoria</label>
          <input
            type="text"
            id="nome-categoria"
            bind:value={novaCategoria.nome}
            placeholder="Nova categoria..."
            class="w-full px-3 py-2 bg-gray-700 border border-gray-600 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 text-gray-100"
          >
        </div>
        <div class="w-40">
          <label for="cor-categoria" class="sr-only">Cor da Categoria</label>
          <select
            id="cor-categoria"
            bind:value={novaCategoria.cor}
            class="w-full px-3 py-2 bg-gray-700 border border-gray-600 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 text-gray-100"
          >
            {#each cores as cor}
              <option value={cor.valor}>{cor.nome}</option>
            {/each}
          </select>
        </div>
        <button
          type="submit"
          class="px-4 py-2 bg-gradient-to-r from-indigo-500 to-indigo-600 text-white rounded-lg font-medium hover:from-indigo-600 hover:to-indigo-700 transition-colors"
        >
          Adicionar
        </button>
      </div>
    </form>

    <div class="space-y-3">
      {#if categorias.length === 0}
        <div class="bg-gray-800/50 rounded-xl p-6 text-center border border-gray-600">
          <svg xmlns="http://www.w3.org/2000/svg" class="h-12 w-12 mx-auto text-gray-400 mb-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 7h.01M7 3h5c.512 0 1.024.195 1.414.586l7 7a2 2 0 010 2.828l-7 7a2 2 0 01-2.828 0l-7-7A1.994 1.994 0 013 12V7a4 4 0 014-4z" />
          </svg>
          <p class="text-gray-300 text-lg font-medium mb-2">Nenhuma categoria</p>
          <p class="text-gray-400">Adicione sua primeira categoria usando o formulário acima</p>
        </div>
      {:else}
        {#each categorias as categoria (categoria.id)}
          <div class="bg-gray-800/50 rounded-xl p-4 border border-gray-600">
            <div class="flex items-center justify-between">
              <div class="flex items-center gap-3">
                <div class="w-3 h-3 rounded-full {getCorClasse(categoria.cor)}"></div>
                <span class="text-gray-200">{categoria.nome}</span>
              </div>
              <div class="flex gap-2">
                <button
                  on:click={() => handleEdit(categoria)}
                  class="p-2 text-gray-400 hover:text-indigo-400 transition-colors"
                  title="Editar categoria"
                >
                  <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                    <path d="M13.586 3.586a2 2 0 112.828 2.828l-.793.793-2.828-2.828.793-.793zM11.379 5.793L3 14.172V17h2.828l8.38-8.379-2.83-2.828z" />
                  </svg>
                </button>
                <button
                  on:click={() => handleDelete(categoria)}
                  class="p-2 text-gray-400 hover:text-red-400 transition-colors"
                  title="Excluir categoria"
                >
                  <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                    <path fill-rule="evenodd" d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z" clip-rule="evenodd" />
                  </svg>
                </button>
              </div>
            </div>
          </div>
        {/each}
      {/if}
    </div>
  </div>
</div>

{#if showModal}
  <div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50">
    <div class="bg-gray-800 rounded-xl p-6 max-w-md w-full">
      <h3 class="text-xl font-semibold text-gray-200 mb-4">
        {modalAction === 'edit' ? 'Editar categoria' : 'Excluir categoria'}
      </h3>
      
      {#if modalAction === 'edit'}
        <div class="space-y-4 mb-6">
          <div>
            <label for="edit-nome" class="block text-sm font-medium text-gray-300 mb-1">Nome da Categoria</label>
            <input
              type="text"
              id="edit-nome"
              bind:value={categoriaEditando.nome}
              class="w-full px-3 py-2 bg-gray-700 border border-gray-600 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 text-gray-100"
            >
          </div>
          <div>
            <label for="edit-cor" class="block text-sm font-medium text-gray-300 mb-1">Cor</label>
            <select
              id="edit-cor"
              bind:value={categoriaEditando.cor}
              class="w-full px-3 py-2 bg-gray-700 border border-gray-600 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 text-gray-100"
            >
              {#each cores as cor}
                <option value={cor.valor}>{cor.nome}</option>
              {/each}
            </select>
          </div>
        </div>
      {:else}
        <p class="text-gray-300 mb-6">
          Tem certeza que deseja excluir a categoria "{categoriaEditando.nome}"? Esta ação não pode ser desfeita.
        </p>
      {/if}
      
      <div class="flex justify-end gap-3">
        <button
          on:click={closeModal}
          class="px-4 py-2 text-gray-300 hover:text-gray-100 transition-colors"
        >
          Cancelar
        </button>
        <button
          on:click={confirmAction}
          class="px-4 py-2 bg-gradient-to-r from-indigo-500 to-indigo-600 text-white rounded-lg font-medium hover:from-indigo-600 hover:to-indigo-700 transition-colors"
        >
          {modalAction === 'edit' ? 'Salvar' : 'Excluir'}
        </button>
      </div>
    </div>
  </div>
{/if}
