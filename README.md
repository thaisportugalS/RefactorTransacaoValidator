# Refatoração do Código: `TransacaoValidator`

Neste projeto foi realizada a refatoração de um validador de transações, `TransacaoValidator`, responsável pela validação dos campos de uma transação no formato ISO. A refatoração foi feita com o objetivo de melhorar a legibilidade, clareza e a estrutura geral do código, seguindo os princípios de **Clean Code**.

## Objetivos da Refatoração

- **Melhorar a legibilidade** do código, utilizando nomes de variáveis e métodos mais significativos.
- **Reduzir a complexidade** e o aninhamento de blocos de decisão, para facilitar a compreensão.
- **Evitar código morto** (como blocos `catch` vazios).
- **Melhorar o tratamento de exceções**, para garantir que erros sejam registrados corretamente.
- **Refatoração de métodos** para deixar o código modular, com funções pequenas e reutilizáveis.

## O Código Original

O código original apresentava vários problemas relacionados ao **Clean Code**:

- **Nomes de variáveis e métodos não claros**: Algumas variáveis e funções não expressavam claramente seu propósito.
- **Condicional excessivamente aninhada**: A validação da transação possuía muitos `if` dentro de outros `if`, o que dificultava a compreensão e a manutenção.
- **Bloco `catch` vazio**: A captura de exceções não fazia nada, o que poderia mascarar problemas e dificultar o diagnóstico de erros.
- **Falta de modularização**: A lógica de validação estava centralizada em um único método, o que tornava o código menos reutilizável e difícil de testar.

## Refatoração Realizada

A refatoração foi focada em melhorar os pontos citados. Abaixo estão as mudanças principais:

### 1. Renomeação de Variáveis e Métodos

Os nomes das variáveis e métodos foram alterados para expressar de forma mais clara seus objetivos:

- **`isNotPreenchido` → `isBit02NaoPreenchido`**: O nome foi alterado para refletir que se refere especificamente à verificação do campo `BIT_02`.
- **`validateAux` → `isBit02Vazio`**: Renomeado para indicar claramente que verifica se o valor de `BIT_02` está vazio.
- **`auxValidacao` → `isAuxValidacaoNecessario`**: O nome foi alterado para refletir que essa variável indica se a validação adicional é necessária.
- **Método `salvar` → `salvarTransacao`**: O nome foi alterado para especificar que o método é responsável por salvar uma transação.

### 2. Extração de Lógica para Métodos Auxiliares

A lógica complexa foi dividida em métodos auxiliares para facilitar o entendimento e promover a reutilização:

- **Método `isCamposInvalidos`**: Verifica se os campos essenciais (como `BIT_02`) estão preenchidos corretamente. Essa validação foi isolada para reduzir a complexidade do método `validate`.
- **Método `isProcessamentoValido`**: Agrupa as verificações relacionadas ao processamento da transação, garantindo que os campos necessários estejam presentes e válidos. Isso melhora a clareza do fluxo de validação.

### 3. Tratamento de Exceções Melhorado

O bloco `catch` vazio foi removido e substituído por um tratamento adequado de exceções:

- Agora, quando uma exceção é capturada, ela é registrada com o logger (`LOGGER.error`), e uma nova exceção é lançada, garantindo que o erro seja visível e fácil de diagnosticar.

### 4. Simplificação da Lógica Condicional

A lógica de validação foi simplificada:

- As múltiplas verificações e condições foram agrupadas em métodos, o que diminui o número de níveis de aninhamento e facilita a compreensão do fluxo de controle.

## Código Refatorado

Aqui está o código refatorado após as mudanças:

```java
public class TransacaoValidator {

    private static final Logger LOGGER = LoggerFactory.getLogger(TransacaoValidator.class);
    private static final List<String> TIPOS_VALIDOS = List.of("02", "03", "04", "05", "12");

    public void validate(ISOModel modelo) {
        LOGGER.info("Início da validação");

        boolean isBit02NaoPreenchido = modelo.getBit02() == null;
        boolean isBit02Vazio = modelo.getBit02() != null && modelo.getBit02().getValue().isEmpty();
        boolean isAuxValidacaoNecessario = isBit02Vazio && modelo.getBit03() == null;
        String valorDeValidacao = isBit02NaoPreenchido ? "01" : "02";

        if (isCamposInvalidos(isBit02NaoPreenchido, isBit02Vazio, isAuxValidacaoNecessario, valorDeValidacao)) {
            throw new IllegalArgumentException("Valores não preenchidos corretamente");
        }

        try {
            if (isProcessamentoValido(modelo)) {
                salvarTransacao(modelo, isAuxValidacaoNecessario);
            }
        } catch (Exception e) {
            LOGGER.error("Erro ao processar a transação", e);
            throw new IllegalArgumentException("Erro no processamento da transação", e);
        }
    }

    private boolean isCamposInvalidos(boolean isBit02NaoPreenchido, boolean isBit02Vazio, boolean isAuxValidacaoNecessario, String valorDeValidacao) {
        return isBit02NaoPreenchido || isBit02Vazio && !isAuxValidacaoNecessario && valorDeValidacao.equals("01");
    }

    private boolean isProcessamentoValido(ISOModel modelo) {
        return modelo.getBit03() != null && modelo.getBit04() != null
                && TIPOS_VALIDOS.contains(modelo.getBit04().getValue())
                && modelo.getBit05() != null && modelo.getBit12() != null;
    }

    private void salvarTransacao(ISOModel modelo, boolean isAuxValidacaoNecessario) {
        if (isAuxValidacaoNecessario) {
            throw new IllegalArgumentException("Validação falhou");
        }

        LOGGER.info("Salvando transação com o valor de BIT_02: {}", modelo.getBit02().getValue());
    }
}
