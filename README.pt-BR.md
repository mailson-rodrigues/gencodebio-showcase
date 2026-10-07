# GenCodeBio

**Plataforma web integrada de bioinformática para genômica clínica — do dado bruto de variantes a uma interpretação auditável e baseada em evidências.**

> 🇺🇸 [Read in English](README.md)

> **Status:** em desenvolvimento ativo · **Research Use Only (RUO)** · nenhum dado real de paciente é processado.
> Este repositório é uma **vitrine pública** (arquitetura, decisões de projeto, material selecionado).
> O código-fonte de produção é privado e proprietário. Veja [LICENSE](LICENSE) e [NOTICE](NOTICE).

## O que funciona hoje

- **Pipeline de variantes:** upload de VCF (parser em streaming) → QC de entrada → anotação com **Ensembl VEP** (transcrito canônico / MANE Select) → enriquecimento com **ClinVar**, **gnomAD**, **PubMed** e preditores in silico (**REVEL, AlphaMissense, CADD**).
- **Classificação ACMG/AMP assistida por IA**, com regra rígida: *resultado sem citação de fonte válida e verificável é descartado e nunca persistido.*
- **Painéis virtuais curados**, **laudo em PDF**, busca e filtro de variantes, upload de arquivos grandes por URL assinada, processamento assíncrono em lotes.
- **Segurança e governança:** sessões revogáveis no servidor, RBAC, MFA TOTP, cadastro por convite com aprovação, rate limiting, trilha de auditoria completa.
- **Qualidade:** 279 testes automatizados no backend, CI a cada push, auditoria de dependências, histórico documentado por etapas.

## Arquitetura e roadmap

Veja o diagrama e os princípios (portas e adaptadores, proveniência, evidência antes de eloquência, conformidade por construção) na versão em [inglês](README.md) e em [docs/architecture.md](docs/architecture.md).

## Autor

**Mailson Rodrigues** — biomédico e desenvolvedor, na interseção entre genética clínica, patologia molecular e engenharia de software. Aberto a conversas com investidores, colaboradores e equipes de genômica e saúde digital.

## Licença

Uso comercial, redistribuição e uso deste material para construir produtos concorrentes **não são permitidos** — veja [LICENSE](LICENSE).

© 2026 Mailson Rodrigues. Todos os direitos reservados, exceto conforme a licença.
