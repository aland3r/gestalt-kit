---
slug: deviante-suporte-decisao-manutencao
product: deviante
venue: scientific
status: published
published_at: 2026-08-04
read_minutes: 4
title: Sistema de suporte à decisão para manutenção industrial
excerpt: Desenvolvimento de um sistema web que combina IPDD, ADWIN, aprendizado de máquina e avaliação humana para detectar desvios e apoiar a manutenção baseada em condição.
---

O trabalho narra o desenvolvimento de um sistema de suporte à decisão em gestão de manutenção industrial. O produto de software origina-se do estudo e implantação do IPDD, um framework que recebe dados de máquinas e detecta pontos próximos ao início da degradação através de previsões facilitadas por aprendizado de máquina, permitindo a gestores direcionar anomalias para ações corretivas de inspeção e manutenção baseada em condição. A identificação estatística das mudanças cabe ao algoritmo de janelamento adaptativo ADWIN, que compara registros antigos e recentes em janelas ajustadas dinamicamente, sinalizando desvio quando a diferença entre elas ultrapassa um limiar estatístico definido por um parâmetro de sensibilidade configurável através de componentes da interface. Análises são geradas tanto sobre o tempo de permanência nos eventos quanto a parâmetros de condição do maquinário. Porém, as anomalias detectadas ainda exigem o ponderamento do gestor para a tomada de decisão. A detecção automatizada integrada à avaliação humana atende a princípios de centralidade humana previstos na Indústria 5.0. Partimos de uma lacuna encontrada na literatura: a ausência de estrutura hierárquica entre objetos e atributos do domínio de manutenção gera sobrecarga cognitiva para a interpretação. Foi aplicado o processo ORCA, da metodologia de OOUX, para estruturar objetos de valor do ecossistema de manutenção. O processo resultou em seis objetos de domínio que, aliados a nove casos de uso, compuseram a usabilidade da interface em ambiente web, desde o carregamento de dados até a execução de análises de desvio e o agendamento de ações proativas. O trabalho visa contribuir com a interação de parâmetros que devem ser analisados e ajustados dinamicamente, convergindo na interface o diagnóstico do equipamento, informações do processo e falhas detectadas. A estruturação semântica do domínio, aliada a requisitos funcionais especificado no caso de uso, foi o que viabilizou a implantação do sistema por meio de desenvolvimento em linguagem natural, otimizado por um repositório de componentes reutilizáveis de IA. Testes iniciais identificaram desvios em dados sintéticos e apresentaram bons resultados.

Palavras-chave: 1. Mineração de Processo; 2. Manutenção baseada em condição; 3. Experiência Orientada a Objetos; 4. Sistema de Suporte a Decisão.
