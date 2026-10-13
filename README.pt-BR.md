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
- **Internacional:** interface em português, inglês, espanhol e francês, escolhida no acesso; erros da API com códigos estáveis traduzidos no frontend; o CI falha se faltar uma tradução.
- **Perfil local de custo zero** *(implementado no repositório)*: portas e adaptadores para fila de jobs, armazenamento de objetos, e-mail e LLM, escolhidos por perfil de execução. Um único `docker compose up` sobe PostgreSQL migrado, armazenamento compatível com S3, API e interface em um notebook, sem custo de nuvem. Localmente a classificação por IA vem desligada e, quando desligada, a API informa isso de forma explícita em vez de devolver um resultado sem suporte.
- **Dados de referência locais** *(implementado no repositório)*: o cache oficial do Ensembl VEP como serviço local (equivalência REST × local conferida em 330 de 330 variantes), um snapshot versionado do ClinVar casado pelo **alelo** (não pelo rsID) e fatias regionais do gnomAD v4.1 para os 33 genes dos painéis curados. Uma regra de cobertura transforma «fora das regiões extraídas» em falha técnica explícita — nunca uma falsa «ausência no gnomAD». Na validação do snapshot do ClinVar medimos que a consulta pública por rsID devolveu o registro de outro alelo em 66 de 129 rsIDs com mais de um registro; o backend local corrige isso por construção.
- **Proveniência e reinterpretação** *(implementado no repositório)*: todo laudo em PDF traz uma seção de rastreabilidade (imagem de software, versões e datas das bases, modelo e hash do prompt), e um endpoint de leitura compara a evidência gravada com o snapshot local atual e sinaliza classificações possivelmente defasadas. Nunca reclassifica sozinho.
- **Cliente de linha de comando** *(implementado no repositório)*: `gencodebio` — acesso com MFA, envio de VCF, execução das etapas, listagem de variantes, download do laudo e consulta da proveniência. A senha nunca vai na linha de comando; só o token de sessão é guardado (permissão restrita ao dono) e o logout revoga a sessão no servidor. Usa apenas a API pública.
- **Operação** *(implementado no repositório)*: endpoint de prontidão que verifica o banco (o health check simples deliberadamente não verifica), geração de SBOM e varredura de vulnerabilidades da imagem no CI.
- **Qualidade:** 585 testes automatizados no backend (também executados contra um PostgreSQL real no CI), 42 no frontend e 26 na CLI; CI a cada push, auditoria de dependências como gate, varredura OWASP ZAP e histórico documentado por etapas.

> **Nota de versão:** os itens marcados *implementado no repositório* estão construídos e testados, mas ainda não foram publicados na instância hospedada, que será atualizada em uma versão planejada.

## Arquitetura e roadmap

Veja o diagrama e os princípios (portas e adaptadores, proveniência, evidência antes de eloquência, conformidade por construção) na versão em [inglês](README.md) e em [docs/architecture.md](docs/architecture.md).

## Autor

**Mailson Rodrigues** — biomédico e desenvolvedor, na interseção entre genética clínica, patologia molecular e engenharia de software. Aberto a conversas com investidores, colaboradores e equipes de genômica e saúde digital.

## Licença

Uso comercial, redistribuição e uso deste material para construir produtos concorrentes **não são permitidos** — veja [LICENSE](LICENSE).

© 2026 Mailson Rodrigues. Todos os direitos reservados, exceto conforme a licença.
