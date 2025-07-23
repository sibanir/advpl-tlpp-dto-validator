# ADVPL DTO Validator

Exemplo simples validante DTOs (Data Transfer Objects) em ADVPL/TLPP, permitindo definição declarativa de regras de validação para APIs REST e outras integrações.

## Funcionalidades

- ✅ Validação de tipos de dados (caractere, numérico, data, array, JSON)
- ✅ Validações específicas por tipo (min/max, regex, intervalos)
- ✅ Suporte para estruturas JSON aninhadas e arrays
- ✅ Mensagens de erro descritivas e contextuais
- ✅ Facilmente integrável a APIs REST e microsserviços
- ✅ Validação automática baseada no dicionário de dados do Protheus (SX3)

## Instalação

Clone o repositório ou copie os arquivos para seu projeto:

```bash
git clone https://github.com/sibanir/advpl-tlpp-dto-validator.git
```

## Uso Básico

```advpl
#INCLUDE "TOTVS.CH"

// Criar o validador
oValidator := DTOValidator():new()

// Definir regras de validação
jRules := {;
    "name": { "type": "C", "required": .T., "min": 3 },;
    "age": { "type": "N", "required": .T., "min": 18 };
}

// Dados a serem validados
jData := {;
    "name": "João Silva",;
    "age": 25;
}

// Realizar validação
lValid := oValidator:validateDTOStructure(jData, jRules)

If !lValid
    ConOut("Erro: " + oValidator:errorMessage)
EndIf
```

### Validação por Dicionário de Dados

```advpl
#INCLUDE "TOTVS.CH"

// Criar o validador
oValidator := DTOValidator():new()

// Dados a serem validados
jData := {;
    "a1_cod": "000001",;
    "a1_nome": "João Silva",;
    "a1_nreduz": "J.Silva";
}

// Validar usando dicionário - todos os campos da tabela
lValid := oValidator:validateByDictionary(jData, 'SA1')

// Validar usando dicionário - campos específicos
lValid := oValidator:validateByDictionary(jData, 'SA1', {'A1_COD', 'A1_NOME', 'A1_NREDUZ'})

If !lValid
    ConOut("Erro: " + oValidator:errorMessage)
EndIf
```

## Regras de Validação

A validação é baseada em um objeto JSON que define regras para cada campo:

| Propriedade | Descrição | Aplicável a |
|-------------|-----------|-------------|
| `type` | Tipo de dado (`C`, `N`, `D`, `L`, `A`, `J`) | Todos |
| `required` | Se o campo é obrigatório | Todos |
| `min` | Valor/comprimento mínimo | `C`, `N` |
| `max` | Valor/comprimento máximo | `C`, `N` |
| `pattern` | Padrão regex para validação | `C` |
| `default` | Valor padrão se não informado | Todos |
| `valueList` | Lista de valores permitidos | `C`, `N` |
| `fields` | Campos aninhados para JSON | `J` |
| `items` | Definição para itens de array | `A` |

## Exemplos Avançados

### Validação de Array

```advpl
jRules := {;
    "products": {;
        "type": "A",;
        "required": .T.,;
        "items": {;
            "type": "C",;
            "min": 3,;
            "max": 20;
        };
    };
}
```

### Validação de JSON Aninhado

```advpl
jRules := {;
    "customer": {;
        "type": "J",;
        "required": .T.,;
        "fields": {;
            "id": { "type": "C", "required": .T. },;
            "name": { "type": "C", "required": .T. },;
            "active": { "type": "L", "required": .F., "default": .T. };
        };
    };
}
```

### Validação de Array de Objetos

```advpl
jRules := {;
    "items": {;
        "type": "A",;
        "required": .T.,;
        "items": {;
            "type": "J",;
            "fields": {;
                "productId": { "type": "C", "required": .T. },;
                "quantity": { "type": "N", "required": .T., "min": 1 };
            };
        };
    };
}
```

## Uso em APIs REST

Para integrar facilmente a validação em APIs REST:

```advpl
// No controller da API - Validação com DTO customizado
Method post() Class YourController
    Local jData := JsonObject():NewFromJson(oRest:GetBodyRequest())
    Local oDTO := YourDTO():new()
    Local oValidator := DTOValidator():new()
    
    If !oValidator:validateDTOStructure(jData, oDTO:validationRules)
        oRest:SetStatus(400)
        oRest:SetResponse({"error": .T., "message": oValidator:errorMessage})
        Return .F.
    EndIf
    
    // Processamento normal...
Return .T.

// No controller da API - Validação com dicionário
Method post() Class CustomerController
    Local jData := JsonObject():NewFromJson(oRest:GetBodyRequest())
    Local oValidator := DTOValidator():new()
    
    // Valida usando estrutura da tabela SA1
    If !oValidator:validateByDictionary(jData, 'SA1', {'A1_COD', 'A1_NOME', 'A1_TIPO'})
        oRest:SetStatus(400)
        oRest:SetResponse({"error": .T., "message": oValidator:errorMessage})
        Return .F.
    EndIf
    
    // Processamento normal...
Return .T.
```

## Definindo DTOs

Criar classes específicas para cada DTO:

```advpl
Class ProductDTO
    Public Data validationRules as json
    Public Method new() Constructor
EndClass

Method new() Class ProductDTO
    // Usando JsonObject para melhor organização
    Local jItemFields := JsonObject():New()
    
    jItemFields["productId"] := {"type": "C", "required": .T.}
    jItemFields["quantity"] := {"type": "N", "required": .T., "min": 1}
    
    ::validationRules := JsonObject():New()
    ::validationRules["orderId"] := {"type": "C", "required": .T.}
    ::validationRules["items"] := {"type": "A", "required": .T., "items": {"type": "J", "fields": jItemFields}}
Return(Self)
```
