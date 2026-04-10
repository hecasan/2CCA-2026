# Aula 10/04/2026 — Refatoração e Componentização do Sistema de Consultas Mobile

## Objetivo da Aula

Na aula anterior (16/03/2026) criamos a estrutura de tipos do domínio do sistema e integramos a modelagem TypeScript ao aplicativo mobile.
Ao final daquela aula, todo o código visual e lógica da aplicação estavam concentrados no arquivo `App.tsx`.

Hoje iremos evoluir a arquitetura do projeto aplicando conceitos importantes de componentização e separação de responsabilidades.

**Objetivos específicos da aula:**

- Criar componentes reutilizáveis separados
- Implementar a camada de Screens (telas)
- Separar estilos em arquivos dedicados
- Criar exports centralizados para melhor organização
- Refatorar o `App.tsx` para uma estrutura mais profissional

---

## Por que essa refatoração é importante?

No momento, nosso `App.tsx` está fazendo **tudo**:

- Definindo dados
- Gerenciando estado
- Renderizando interface
- Contendo todos os estilos

Em projetos reais, essa abordagem não escala bem. Hoje aprenderemos a organizar o código seguindo as melhores práticas do React Native.

---

## 1. Atualizar o Repositório

Antes de começar, certifique-se de que seu código local está sincronizado com o repositório remoto.

```bash
git clone SEU-REPOSITÓRIO
```

---

## 2. Nova Estrutura de Pastas

Vamos criar novas pastas dentro de `src` para organizar melhor nosso projeto.

### 2.1 Criar a pasta `components`

No VS Code:

1. Clique com botão direito sobre a pasta `src`
2. Escolha **New Folder**
3. Digite: `components`

### 2.2 Criar a pasta `screens`

Repita o processo:

1. Clique com botão direito sobre a pasta `src`
2. Escolha **New Folder**
3. Digite: `screens`

### 2.3 Criar a pasta `styles`

Mais uma vez:

1. Clique com botão direito sobre a pasta `src`
2. Escolha **New Folder**
3. Digite: `styles`

**Estrutura esperada até aqui:**

```
src
├── components    (nova)
├── screens       (nova)
├── styles        (nova)
├── types
└── interfaces
```

---

## 3. Separando os Estilos

Vamos começar extraindo os estilos que estão no `App.tsx` para arquivos separados.

### 3.1 Criar o arquivo de estilos do App

Dentro da pasta `styles`, crie o arquivo **`app.styles.ts`**:

```typescript
import { StyleSheet } from "react-native";

export const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: "#79059C",
  },
  scrollContent: {
    padding: 20,
    paddingTop: 60,
  },
  header: {
    alignItems: "center",
    marginBottom: 24,
  },
  titulo: {
    fontSize: 28,
    fontWeight: "bold",
    color: "#fff",
    marginBottom: 8,
  },
  subtitulo: {
    fontSize: 18,
    color: "#fff",
    opacity: 0.9,
  },
});
```

> **Explicação:** Este arquivo contém os estilos relacionados à estrutura geral da aplicação:
> - `container`: layout principal com fundo roxo
> - `scrollContent`: padding da área de scroll
> - `header`: cabeçalho centralizado
> - `titulo` e `subtitulo`: estilos do texto do cabeçalho

### 3.2 Criar o arquivo de estilos do Card de Consulta

Dentro da pasta `styles`, crie o arquivo **`consultaCard.styles.ts`**:

```typescript
import { StyleSheet } from "react-native";

export const styles = StyleSheet.create({
  card: {
    backgroundColor: "#fff",
    borderRadius: 16,
    padding: 20,
    shadowColor: "#000",
    shadowOffset: { width: 0, height: 4 },
    shadowOpacity: 0.2,
    shadowRadius: 8,
    elevation: 5,
  },
  statusBadge: {
    backgroundColor: "#FFA500",
    alignSelf: "flex-start",
    paddingHorizontal: 16,
    paddingVertical: 8,
    borderRadius: 20,
    marginBottom: 20,
  },
  statusConfirmada: {
    backgroundColor: "#4CAF50",
  },
  statusCancelada: {
    backgroundColor: "#F44336",
  },
  statusTexto: {
    color: "#fff",
    fontWeight: "bold",
    fontSize: 12,
  },
  secao: {
    marginBottom: 20,
    paddingBottom: 20,
    borderBottomWidth: 1,
    borderBottomColor: "#e0e0e0",
  },
  label: {
    fontSize: 16,
    fontWeight: "bold",
    color: "#79059C",
    marginBottom: 8,
  },
  valor: {
    fontSize: 18,
    color: "#333",
    marginBottom: 4,
  },
  info: {
    fontSize: 14,
    color: "#666",
    marginBottom: 2,
  },
  observacoes: {
    fontSize: 14,
    color: "#555",
    fontStyle: "italic",
    marginTop: 8,
  },
  acoes: {
    marginTop: 10,
  },
  botaoContainer: {
    marginBottom: 12,
  },
  mensagem: {
    backgroundColor: "#E8F5E9",
    padding: 16,
    borderRadius: 8,
    borderLeftWidth: 4,
    borderLeftColor: "#4CAF50",
  },
  mensagemCancelada: {
    backgroundColor: "#FFEBEE",
    padding: 16,
    borderRadius: 8,
    borderLeftWidth: 4,
    borderLeftColor: "#F44336",
  },
  mensagemTexto: {
    fontSize: 16,
    color: "#333",
    fontWeight: "600",
    textAlign: "center",
  },
});
```

> **Explicação:** Este arquivo contém todos os estilos relacionados à exibição do card de consulta: estilos do card, badge de status, seções de informação, botões de ação e mensagens de feedback.

---

## 4. Criando o Componente ConsultaCard

Agora vamos extrair a lógica de exibição de uma consulta para um componente separado.

Dentro da pasta `components`, crie o arquivo **`ConsultaCard.tsx`**:

```tsx
import React from "react";
import { View, Text, Button } from "react-native";
import { Consulta } from "../interfaces/consulta";
import { styles } from "../styles/consultaCard.styles";

type ConsultaCardProps = {
  consulta: Consulta;
  onConfirmar?: () => void;
  onCancelar?: () => void;
};

export default function ConsultaCard({
  consulta,
  onConfirmar,
  onCancelar,
}: ConsultaCardProps) {
  function formatarValor(valor: number): string {
    return valor.toLocaleString("pt-BR", {
      style: "currency",
      currency: "BRL",
    });
  }

  function formatarData(data: Date): string {
    return data.toLocaleDateString("pt-BR");
  }

  return (
    <View style={styles.card}>
      <View
        style={[
          styles.statusBadge,
          consulta.status === "confirmada" && styles.statusConfirmada,
          consulta.status === "cancelada" && styles.statusCancelada,
        ]}
      >
        <Text style={styles.statusTexto}>
          {consulta.status.toUpperCase()}
        </Text>
      </View>

      <View style={styles.secao}>
        <Text style={styles.label}>👨‍⚕️ Médico</Text>
        <Text style={styles.valor}>{consulta.medico.nome}</Text>
        <Text style={styles.info}>CRM: {consulta.medico.crm}</Text>
        <Text style={styles.info}>{consulta.medico.especialidade.nome}</Text>
      </View>

      <View style={styles.secao}>
        <Text style={styles.label}>👤 Paciente</Text>
        <Text style={styles.valor}>{consulta.paciente.nome}</Text>
        <Text style={styles.info}>CPF: {consulta.paciente.cpf}</Text>
        <Text style={styles.info}>Email: {consulta.paciente.email}</Text>
        {consulta.paciente.telefone && (
          <Text style={styles.info}>Tel: {consulta.paciente.telefone}</Text>
        )}
      </View>

      <View style={styles.secao}>
        <Text style={styles.label}>📅 Dados da Consulta</Text>
        <Text style={styles.valor}>Data: {formatarData(consulta.data)}</Text>
        <Text style={styles.valor}>
          Valor: {formatarValor(consulta.valor)}
        </Text>
        {consulta.observacoes && (
          <Text style={styles.observacoes}>{consulta.observacoes}</Text>
        )}
      </View>

      <View style={styles.acoes}>
        {consulta.status === "agendada" && (
          <>
            {onConfirmar && (
              <View style={styles.botaoContainer}>
                <Button
                  title="Confirmar Consulta"
                  onPress={onConfirmar}
                  color="#4CAF50"
                />
              </View>
            )}
            {onCancelar && (
              <View style={styles.botaoContainer}>
                <Button
                  title="Cancelar Consulta"
                  onPress={onCancelar}
                  color="#F44336"
                />
              </View>
            )}
          </>
        )}

        {consulta.status === "confirmada" && (
          <View style={styles.mensagem}>
            <Text style={styles.mensagemTexto}>
              ✓ Consulta confirmada com sucesso!
            </Text>
          </View>
        )}

        {consulta.status === "cancelada" && (
          <View style={styles.mensagemCancelada}>
            <Text style={styles.mensagemTexto}>✗ Consulta cancelada</Text>
          </View>
        )}
      </View>
    </View>
  );
}
```

> **Explicação:** Este componente é responsável apenas por exibir os dados de uma consulta.
>
> **Props do componente:**
> - `consulta`: objeto do tipo `Consulta` que será exibido
> - `onConfirmar`: função opcional chamada quando o usuário confirmar
> - `onCancelar`: função opcional chamada quando o usuário cancelar
>
> **Por que usar `?` nas props `onConfirmar` e `onCancelar`?**
> O `?` indica que essas props são opcionais. Isso significa que o componente pode ser usado sem passar essas funções, tornando-o mais flexível.
>
> **Vantagens dessa separação:**
> - O componente pode ser reutilizado em várias telas
> - A lógica de exibição está isolada
> - Facilita testes e manutenção
> - Deixa o código mais organizado

---

## 5. Criando a Tela Home

Agora vamos criar nossa primeira "tela" (screen) do aplicativo.

Dentro da pasta `screens`, crie o arquivo **`Home.tsx`**:

```tsx
import React, { useState } from "react";
import { View, Text, ScrollView } from "react-native";
import { StatusBar } from "expo-status-bar";
import { Especialidade } from "../types/especialidade";
import { Paciente } from "../types/paciente";
import { Medico } from "../interfaces/medico";
import { Consulta } from "../interfaces/consulta";
import { ConsultaCard } from "../components";
import { styles } from "../styles/app.styles";

export default function Home() {
  const cardiologia: Especialidade = {
    id: 1,
    nome: "Cardiologia",
    descricao: "Cuidados com o coração",
  };

  const medico1: Medico = {
    id: 1,
    nome: "Dr. Roberto Silva",
    crm: "CRM12345",
    especialidade: cardiologia,
    ativo: true,
  };

  const paciente1: Paciente = {
    id: 1,
    nome: "Carlos Andrade",
    cpf: "123.456.789-00",
    email: "carlos@email.com",
    telefone: "(11) 98765-4321",
  };

  const [consulta, setConsulta] = useState<Consulta>({
    id: 1,
    medico: medico1,
    paciente: paciente1,
    data: new Date(2026, 2, 10),
    valor: 350,
    status: "agendada",
    observacoes: "Consulta de rotina",
  });

  function confirmarConsulta() {
    setConsulta({
      ...consulta,
      status: "confirmada",
    });
  }

  function cancelarConsulta() {
    setConsulta({
      ...consulta,
      status: "cancelada",
    });
  }

  return (
    <View style={styles.container}>
      <StatusBar style="light" />
      <ScrollView contentContainerStyle={styles.scrollContent}>
        <View style={styles.header}>
          <Text style={styles.titulo}>Sistema de Consultas</Text>
          <Text style={styles.subtitulo}>Consulta #{consulta.id}</Text>
        </View>
        <ConsultaCard
          consulta={consulta}
          onConfirmar={confirmarConsulta}
          onCancelar={cancelarConsulta}
        />
      </ScrollView>
    </View>
  );
}
```

> **Explicação:** Esta é a tela principal do aplicativo.
>
> **Responsabilidades da tela `Home`:**
> - Gerenciar o estado da consulta
> - Criar os dados de exemplo (médico, paciente, especialidade)
> - Implementar as funções de confirmar e cancelar
> - Renderizar a estrutura visual (cabeçalho + card)
> - Passar props para o componente `ConsultaCard`
>
> Observe que a lógica de gerenciamento de estado está aqui, e a exibição visual do card está delegada ao componente `ConsultaCard`. Os estilos são importados de `app.styles.ts`.

---

## 6. Criando Exports Centralizados

Para facilitar as importações no projeto, vamos criar arquivos `index.ts` que centralizam os exports.

### 6.1 Export centralizado de componentes

Dentro da pasta `components`, crie o arquivo **`index.ts`**:

```typescript
/**
 * Exportação centralizada dos componentes
 *
 * Permite importar assim:
 * import { ConsultaCard } from './src/components';
 *
 * Em vez de:
 * import ConsultaCard from './src/components/ConsultaCard';
 */

export { default as ConsultaCard } from "./ConsultaCard";

/**
 * Conforme criarmos mais componentes, adicionaremos aqui:
 *
 * export { default as Loading } from "./Loading";
 * export { default as EmptyState } from "./EmptyState";
 * export { default as Header } from "./Header";
 */
```

> **Explicação:** Este arquivo facilita a importação de componentes.
>
> Sem o `index.ts`:
> ```typescript
> import ConsultaCard from "./src/components/ConsultaCard";
> ```
>
> Com o `index.ts`:
> ```typescript
> import { ConsultaCard } from "./src/components";
> ```
>
> Muito mais limpo e profissional!

### 6.2 Export centralizado de screens

Dentro da pasta `screens`, crie o arquivo **`index.ts`**:

```typescript
export { default as Home } from "./Home";
```

> **Explicação:** Mesmo conceito aplicado às telas. Conforme o projeto cresce e criamos mais telas (Login, Perfil, Histórico etc.), basta adicioná-las aqui.

---

## 7. Refatorando o App.tsx

Agora que temos toda a estrutura componentizada, o `App.tsx` se torna extremamente simples.

Substitua **completamente** o conteúdo do arquivo `App.tsx` por:

```tsx
import React from "react";
import { Home } from "./src/screens";

export default function App() {
  return <Home />;
}
```

> **Explicação:** O `App.tsx` agora tem apenas uma responsabilidade: renderizar a tela inicial.
>
> | | Antes (aula anterior) | Agora |
> |---|---|---|
> | Linhas | ~250 linhas | 6 linhas |
> | Conteúdo | Dados, estado, funções, estilos e renderização | Apenas renderiza `<Home />` |
> | Arquitetura | Monolítica | Escalável |

---

## 8. Testando o Projeto Refatorado

Agora vamos executar o projeto e verificar se tudo está funcionando.

### 8.1 Executar o projeto

No terminal do VS Code:

```bash
npx expo start
```

Depois pressione `w` para abrir no navegador.

### 8.2 Verificar o funcionamento

O aplicativo deve funcionar **exatamente como antes** da refatoração.

Se algo não funcionar:

1. Verifique se todos os arquivos foram criados nas pastas corretas
2. Confira se não há erros de digitação nos imports
3. Consulte o professor

---

## 9. Estrutura Final do Projeto

Após implementar todas as mudanças, a estrutura do projeto deve estar assim:

```
sistema-consultas-mobile
│
├── App.tsx                           (apenas renderiza Home)
├── package.json
├── tsconfig.json
└── src
    ├── components
    │   ├── ConsultaCard.tsx          (componente de exibição)
    │   └── index.ts                  (export centralizado)
    │
    ├── screens
    │   ├── Home.tsx                  (tela principal)
    │   └── index.ts                  (export centralizado)
    │
    ├── styles
    │   ├── app.styles.ts             (estilos gerais)
    │   └── consultaCard.styles.ts    (estilos do card)
    │
    ├── types
    │   ├── especialidade.ts
    │   ├── paciente.ts
    │   └── statusConsulta.ts
    │
    └── interfaces
        ├── consulta.ts
        └── medico.ts
```

---

## 10. Conceitos Aplicados

### 10.1 Componentização

- Separamos a lógica de exibição em componentes reutilizáveis
- Criamos props tipadas para passar dados entre componentes
- Aplicamos o conceito de **Single Responsibility Principle**

### 10.2 Separação de Responsabilidades

| Arquivo | Responsabilidade |
|---|---|
| `App.tsx` | Ponto de entrada — renderiza a tela inicial |
| `Home.tsx` | Gerencia estado e lógica de negócio |
| `ConsultaCard.tsx` | Apenas exibe dados |
| Arquivos de estilos | Apenas definem aparência visual |

### 10.3 Arquitetura Escalável

- Estrutura de pastas clara e organizada
- Exports centralizados facilitam importações
- Fácil adicionar novos componentes e telas

### 10.4 TypeScript + React Native

- Props tipadas com TypeScript
- Interfaces garantem contratos entre componentes
- IntelliSense funciona perfeitamente

---

## 11. Vantagens da Refatoração

| | Antes da Refatoração | Depois da Refatoração |
|---|---|---|
| Organização | ❌ Todo código em um único arquivo | ✅ Código organizado em módulos |
| Reuso | ❌ Difícil reutilizar componentes | ✅ Componentes reutilizáveis |
| Estilos | ❌ Misturados com a lógica | ✅ Separados por responsabilidade |
| Manutenção | ❌ Difícil de manter e escalar | ✅ Fácil de adicionar novas funcionalidades |
| Arquitetura | ❌ Monolítica | ✅ Profissional e escalável |

---

## 12. Salvando as Alterações no Git

Após implementar todas as mudanças, vamos commitar no repositório.

### 12.1 Verificar mudanças

```bash
git status
```

### 12.2 Adicionar arquivos

```bash
git add .
```

### 12.3 Commitar

```bash
git commit -m "Refatorado o componente App movendo a lógica de consulta para a tela inicial e criando estilos separados para o ConsultaCard."
```

### 12.4 Enviar para o GitHub

```bash
git push origin main
```

---

## 13. Próximos Passos

Na próxima aula iremos:

- Implementar navegação entre telas
- Criar uma lista de consultas
- Adicionar mais componentes (`Header`, `Loading`, `EmptyState`)
- Trabalhar com `FlatList` para renderizar listas

---

> Por hoje é isso! 🚀
