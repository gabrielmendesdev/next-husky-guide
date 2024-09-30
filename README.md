## Husky, Lint-Staged e Commitlint para Next.js :)

Essa documentação tem como objetivo auxiliar na criação de uma "casca" padrão para o desenvolvimento de projetos.

## 1 - Husky

Husky é uma ferramenta que usamos para gerenciar Git Hooks de forma fácil. Com Husky, podemos definir os hooks (como scripts) que precisamos. Depois disso, ele assume a responsabilidade de acionar os hooks no momento apropriado de execução.

### Instalar o Husky

Como primeiro passo, vamos instalar e configurar o husky em nosso projeto:

```sh
$ npm install --save-dev husky
```

### husky init (Recomendado)

O comando init simplifica a configuração do husky em um projeto. Ele cria um script pre-commit em .husky/ e atualiza o script prepare no package.json. Modificações podem ser feitas posteriormente para adequar ao seu fluxo de trabalho.

```sh
$ npx husky init
```

## 2 - Lint-Staged

Lint-Staged é uma ferramenta útil para executar linters apenas nos arquivos staged. Sem este pacote, os linters são executados em todo o projeto todas as vezes, e consomem tempo desnecessário para verificar códigos que já foram analisados.

### Instalar lint-staged (Prettier é um adicional)

```sh
$ npm install --save-dev lint-staged prettier
```

### Atualize o objeto scripts dentro de “package.json”

```JSON
"scripts": {
  "dev": "next dev",
  "build": "next build",
  "start": "next start",
  "lint": "next lint",
  "lint-staged": "lint-staged", // Adicione esse script
  "format": "prettier --check .",
  "format:fix": "prettier --write --list-different .",
  "prepare": "husky"
}
```

### Atualize o conteúdo do arquivo “pre-commit”

```sh
npm run lint-staged --concurrent false
npm run build
```

### Crie o arquivo ".lintstagedrc.json" na raiz do projeto

Este é o arquivo de configuração do lint-staged. Ele define os linters para diferentes padrões de arquivos.

De acordo com essas configurações, o prettier e o eslint corrigem automaticamente os problemas de lint. Isso ajuda a economizar o tempo gasto com correções manuais.

```json
{
  "*.{js,jsx,ts,tsx}": ["prettier --write", "eslint --fix", "eslint"],
  "*.{json,md,yml}": ["prettier --write"]
}
```

## 3 - Commitlint

O Commitlint verifica se as mensagens de commit seguem padrões específicos, ajudando a manter um histórico de commits mais consistente e legível, geralmente em conjunto com regras como Conventional Commits.

### Instalando dependências

```sh
$ npm install --save-dev @commitlint/cli @commitlint/config-conventional conventional-changelog-atom
```

### Crie o arquivo “commitlint.config.ts” na raiz do projeto

Este é o arquivo de configuração para conventional-commits. Você deve seguir essas regras ao adicionar um commit.

Você pode alterar as regras conforme necessário. Aqui estão as <a href="https://commitlint.js.org/reference/rules.html">regras disponíveis</a>

```ts
import type { UserConfig } from "@commitlint/types";
import { RuleConfigSeverity } from "@commitlint/types";

const Configuration: UserConfig = {
  extends: ["@commitlint/config-conventional"],
  parserPreset: "conventional-changelog-atom",
  formatter: "@commitlint/format",
  rules: {
    "type-enum": [
      RuleConfigSeverity.Error,
      "always",
      [
        "feat", // New feature
        "fix", // Bug fix
        "docs", // Documentation changes
        "style", // Changes that do not affect the meaning of the code (white-space, formatting, etc.)
        "refactor", // Code changes that neither fix a bug nor add a feature
        "perf", // Performance improvement
        "test", // Adding missing tests or correcting existing tests
        "build", // Changes that affect the build system or external dependencies (example scopes: npm)
        "ci", // Changes to CI configuration files and scripts
        "chore", // Other changes that don't modify src or test files
        "revert", // Reverts a previous commit
      ],
    ],
  },
  // ...
};

export default Configuration;
```

### Atualize o conteúdo do arquivo "tsconfig.json"

```ts
{
  "compilerOptions": {
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [
      {
        "name": "next"
      }
    ],
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": [
    "next-env.d.ts",
    "**/*.ts",
    "**/*.tsx",
    ".next/types/**/*.ts",
    "commitlint.config.ts" // Adicione essa linha
  ],
  "exclude": ["node_modules"]
}
```

### Crie um arquivo dentro da pasta ".husky" chamado "commit-msg"

```sh
$ echo "npx --no -- commitlint --edit \$1" > .husky/commit-msg
```

Parabéns! Você configurou com sucesso a automação de tarefas para otimizar o fluxo de trabalho dos desenvolvedores.

Agora tente adicionar uma mensagem de commit

- Sem adicionar arquivos ao stage
- Sem seguir as regras do commitlint

Se você configurou corretamente o seu ambiente, não será possível adicionar essa mensagem de commit. É necessário primeiro fazer o stage dos arquivos desejados e escrever a mensagem de commit de acordo com as regras do commitlint para realizar um commit com sucesso. Isso evita mensagens de commit sem sentido.

```sh
// Exemplo de uma mensagem válida de commit
feat(setup): add commitlint for commit message validation

// Exemplo de uma mensagem inválida
creating beautiful code with the let's go horse method
```

## Links úteis:

- https://typicode.github.io/husky/get-started.html
- https://www.npmjs.com/package/lint-staged
- https://www.conventionalcommits.org/en/v1.0.0/
- https://githooks.com/
- https://commitlint.js.org/guides/getting-started.html

## Observações

Se por algum acaso você receber as seguintes mensagens após a tentativa de realizar um commit:

```sh
.husky/commit-msg: .husky/commit-msg: cannot execute binary file
husky - commit-msg script failed (code 126)
```

Muito provavelmente seu arquivo "commit-msg" dentro da pasta ".husky" está configurado para codificação de arquivo UTF-16. Experimente mudar para UTF-8 e seu problema será resolvido.
