# Best Route — Repositório Umbrella

Este repositório serve apenas para referência cruzada e documentação dos projetos principais do sistema Best Route:

---

<figure style="display:block; width:80%; max-width:100%; margin:0 auto;">
	<img src="https://github.com/user-attachments/assets/196639bd-e6eb-49e3-a5e4-d04a70885e30" alt="Frontend preview" style="display:block; width:100%; height:auto;" />
	<figcaption style="text-align:center;">Frontend - Planner UI (preview)</figcaption>
</figure>
<br/>
<figure style="display:block; width:80%; max-width:100%; margin:0.75rem auto;">
	<img src="https://github.com/user-attachments/assets/9ddc64c2-d9ff-4547-80d4-f7a23e06eba3" alt="Console with plot view" style="display:block; width:100%; height:auto;" />
	<figcaption style="text-align:center;">Console - plot view</figcaption>
</figure>
<br/>
<figure style="display:block; width:80%; max-width:100%; margin:0.75rem auto;">
	<img src="https://github.com/user-attachments/assets/ccbd54af-157c-4092-af3c-27fa56f25a24" alt="API Swagger UI" style="display:block; width:100%; height:auto;" />
	<figcaption style="text-align:center;">API - Swagger UI (http://localhost:{port}/docs)</figcaption>
</figure>
<br/>
## 🗺️ Diagrama de Arquitetura

Esta seção contém diagramas Mermaid que mostram a arquitetura geral do sistema, o modelo de composição (factories e strategies) e o fluxo do motor genético.

### 1) Arquitetura geral (visão macro)

```mermaid
flowchart TB
	subgraph Entrypoints
		API[FastAPI / API]
		Console[Console]
		Lab[Lab / Benchmarks]
	end

	subgraph Application
		ROSvc[RouteOptimizationService]
	end

	subgraph Infrastructure
		GraphGen[OSMnxGraphGenerator]
		RouteCalc[RouteCalculator]
		BundleFactory[RouteGAExecutionBundle Factory]
		Runner[GeneticAlgorithmExecutionRunner]
		GA[GeneticAlgorithm Engine]
		TSP[TSPGeneticProblem]
	end

	Entrypoints --> ROSvc
	ROSvc --> GraphGen
	ROSvc --> RouteCalc
	ROSvc --> BundleFactory
	BundleFactory --> Runner
	Runner --> GA
	GA --> TSP
	GraphGen --> CachedGeocode[CachedGeocodingResolver]
	RouteCalc --> CachedAdjacency[CachedAdjacencyMatrixBuilder]
```


### 2) Modelo de composição (factories e strategies)

```mermaid
flowchart LR
	Dependencies["Composition root (api/dependencies.py, console/main.py)"] -->|cria| OptimizerFactory["tsp_optimizer_factory"]
	OptimizerFactory -->|resolve| CrossoverFactory["crossover_strategy_factory"]
	OptimizerFactory -->|resolve| MutationFactory["mutation_strategy_factory"]
	OptimizerFactory -->|resolve| SelectionFactory["selection_strategy_factory"]
	OptimizerFactory -->|cria| ExecutionBundle["RouteGAExecutionBundle"]
	ExecutionBundle -->|fornece| Problem["TSPGeneticProblem"]
	ExecutionBundle -->|fornece| StateController["IGeneticStateController"]
	ExecutionBundle -->|fornece| SeedData["RoutePopulationSeedData"]
	StateController -->|injeta| Operators["selection, crossover, mutation, population_generator"]
```

### 3) Decomposição do motor GA (componentes)

```mermaid
flowchart TB
	GAEngine["GeneticAlgorithm Engine"] --> ProblemAdapter["IGeneticProblem / TSPGeneticProblem"]
	GAEngine --> StateController["IGeneticStateController"]
	GAEngine --> Operators["selection, crossover, mutation, pop-gen"]
	GAEngine --> Population["Population Generator"]
	GAEngine --> Evaluator["Problem.evaluate_population"]
	Evaluator --> ProblemAdapter
```

### 4) Fluxo de cálculo de rota (sequence simplified)

```mermaid
sequenceDiagram
	participant Client
	participant API
	participant ROSvc
	participant GraphGen
	participant RouteCalc
	participant Runner
	participant GA
	Client->>API: POST /optimize_route
	API->>ROSvc: optimize(origin, destinations, config)
	ROSvc->>GraphGen: initialize(origin, destinations)
	ROSvc->>RouteCalc: build adjacency matrix
	ROSvc->>Runner: create bundle & run
	Runner->>GA: solve(seed_data, population_size...)
	GA-->>Runner: OptimizationResult
	Runner-->>ROSvc: Result
	ROSvc-->>API: Optimized routes (lat/lon)
```

---
 
## Repositórios do projeto

- **Backend**: [luizaaca/api_best_route](https://github.com/luizaaca/api_best_route)
- **Frontend**: [luizaaca/best_route_tsp_front](https://github.com/luizaaca/best_route_tsp_front)

---

## 📚 Documentação dos Projetos

### Backend — api_best_route

- [`README.md`](https://github.com/luizaaca/api_best_route/blob/main/README.md): Instruções de instalação, execução e visão geral da API.
- [`architecture.md`](https://github.com/luizaaca/api_best_route/blob/main/architecture.md): Arquitetura, camadas e padrões de projeto.
- [`generic_ga.md`](https://github.com/luizaaca/api_best_route/blob/main/generic_ga.md): Descrição do motor genérico de algoritmo genético.
- [`routes_optimization.md`](https://github.com/luizaaca/api_best_route/blob/main/routes_optimization.md): Modelos e pipeline de otimização de rotas.
- [`lab/README.md`](https://github.com/luizaaca/api_best_route/blob/main/lab/README.md): Execuções benchmarks e modo laboratório.
- [`changelog/`](https://github.com/luizaaca/api_best_route/tree/main/changelog): Notas de versão.

### Frontend — best_route_tsp_front

- [`README.md`](https://github.com/luizaaca/best_route_tsp_front/blob/main/README.md): Instruções, overview e exemplos de uso do frontend.
- Outros arquivos e guias: Consulte o repositório para mais detalhes técnicos e exemplos.

---

## 🚀 Como rodar localmente

Clone ambos os projetos:

```bash
git clone https://github.com/luizaaca/api_best_route
git clone https://github.com/luizaaca/best_route_tsp_front
```

Siga as instruções de cada `README.md` para instalar dependências e rodar backend e frontend.

---
