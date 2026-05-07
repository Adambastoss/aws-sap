##### Service Control Policies (SCPs)

Ponto central da prova SAP-C02:

> SCP não concede permissões.  
> SCP apenas define o que PODE ou NÃO PODE ser permitido dentro das contas da organização.

Ou seja:

- **IAM** Policy → concede permissões.
- SCP → limita o teto dessas permissões.

Mesmo que um usuário/role tenha `AdministratorAccess`, uma SCP ainda pode bloquear ações.

# Estrutura do AWS Organizations

As SCPs podem ser aplicadas em:

- Root
- Organizational Units (OUs)
- Contas individuais

Hierarquia:

Root  
├── OU-Prod  
	│ ├── Conta A  
	│ └── Conta B  
└── OU-Dev  
	└── Conta C

As SCPs herdadas da Root + OU + Conta são combinadas.

Se uma OU bloquear algo, a conta abaixo também estará bloqueada.

-----------------------------------------------
# Conceito CRÍTICO para prova

## SCP afeta:

- Usuários IAM
- Roles IAM
- Usuários federados
- Roles assumidas

## SCP NÃO afeta:

### Conta de gerenciamento (management account)

A conta principal do Organizations não é restringida pelas SCPs.

Isso é muito cobrado em pegadinhas.

## 1. Blacklist Strategy (mais comum)

Permite tudo e bloqueia ações específicas.

## 2. Whitelist Strategy

Bloqueia tudo e libera apenas alguns serviços.
**Mais difícil de manter.**

Exemplo:
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:*",
        "s3:*"
      ],
      "Resource": "*"
    }
  ]
}
```

----------------------------------------------------------------
# Exemplos MUITO cobrados na SAP-C02

### Bloquear regiões AWS
Empresa quer permitir apenas:

- us-east-1
- sa-east-1

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyOtherRegions",
      "Effect": "Deny",
      "NotAction": [
        "iam:*",
        "organizations:*",
        "route53:*"
      ],
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": [
            "us-east-1",
            "sa-east-1"
          ]
        }
      }
    }
  ]
}
```

**Por que usar** `NotAction`?

Porque serviços globais:

- IAM
- Route53
- Organizations

não operam em região específica.

Questão clássica da SAP.

### Impedir criação de recursos sem tags

Muito comum em governança enterprise.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyCreateWithoutTag",
      "Effect": "Deny",
      "Action": [
        "ec2:RunInstances"
      ],
      "Resource": "*",
      "Condition": {
        "Null": {
          "aws:RequestTag/Environment": "true"
        }
      }
    }
  ]
}
```

------------------------------
# Cenários típicos de prova

## Cenário 1

“Empresa quer impedir que desenvolvedores desativem CloudTrail em qualquer conta.”

Resposta:

- AWS Organizations + SCP.

## Cenário 2

“Empresa quer impedir criação de recursos fora de regiões aprovadas.”

Resposta:

- SCP com `aws:RequestedRegion`.

## Cenário 3

“Empresa quer garantir que administradores locais das contas não consigam escapar das restrições.”

Resposta:

- SCP explicit deny.