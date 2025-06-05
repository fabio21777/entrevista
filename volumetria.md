## [Volumetria](https://witify.io/en/blog/volumetric-software-planning-and-managing-your-data/)

> **Volumetria:** Quantidade total de dados gerenciados por um sistema (armazenamento, processamento, transmissão).

**Contexto:**
Sistema web para gerenciar eventos e emitir certificados digitais personalizados.

- Geração em lote: cerca de 1.000 certificados por evento
- Tempo médio de geração: 0,2s por certificado (~3min20s por evento)
- Pico de demanda: até 10.000 certificados/dia em períodos de alta
- Necessidade de escalonamento automático das instâncias de geração
