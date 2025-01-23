# Calculadora de Preço por Trabalho

Uma aplicação para calcular preços de projetos baseado no valor/hora, com recursos para gerenciar diferentes tipos de trabalhos e acompanhar seu status.

## Funcionalidades

- 💰 Cálculo automático do valor/hora baseado no salário desejado
- 📊 Gerenciamento de projetos por categorias (Website, Mobile App, Design, etc.)
- 🏷️ Filtros por categoria para visualização rápida
- 📈 Status dos projetos (A Fazer, Feito, Pago, Atrasado)
- 💵 Cálculo automático de impostos e taxas adicionais

## Tecnologias Utilizadas

- Svelte para a interface do usuário
- TailwindCSS para estilização
- LocalStorage para persistência de dados

## Como Usar

1. Clone o repositório
2. Instale as dependências:
```bash
npm install
```

3. Inicie o servidor de desenvolvimento:
```bash
npm run dev
```

4. Acesse `http://localhost:5000` no seu navegador

## Transformando em Aplicativo Desktop

Para criar uma versão desktop executável do aplicativo:

1. Instale as dependências do Electron:
```bash
npm install electron electron-builder --save-dev
```

2. Adicione os scripts no package.json:
```json
{
  "scripts": {
    "electron-dev": "electron .",
    "build": "electron-builder"
  }
}
```

3. Construa o aplicativo:
```bash
npm run build
```

O executável será gerado na pasta `dist`.

## Configurações

### Taxas e Impostos
- Taxa de Urgência: +20%
- Taxa de Complexidade: +30%
- Imposto padrão: 6%

### Cálculo do Valor/Hora
O valor/hora é calculado considerando:
- Salário mensal desejado
- Horas trabalhadas por dia
- Dias trabalhados por semana
- Semanas de férias por ano

## Contribuindo

Sinta-se à vontade para abrir issues ou enviar pull requests com melhorias.

## Licença

MIT
