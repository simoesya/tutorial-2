# Assignment 2 - Mobile Computing Applications

Course: Computação Móvel / Mobile Computing (CM)
Student(s): dam15048
Date: 2026
Repository URL: ___________

---

## 1. Introduction

Este trabalho tem como objetivo o desenvolvimento de competências avançadas em Kotlin e Android, dando continuidade ao Tutorial 1. Foram desenvolvidos três projetos:

- **TP2** — Quatro exercícios Kotlin em consola (sealed classes, generics, DSL com lambda receivers, operator overloading)
- **CoolWeatherApp** — Aplicação Android de meteorologia em tempo real com GPS e suporte multi-layout
- **ImageGalleryApp** — Aplicação Android de galeria de imagens obtidas de uma API pública, desenvolvida com assistência de IA (AntiGravity)

## 2. System Overview

### TP2 — Exercícios Kotlin
Projeto IntelliJ (Maven) com quatro exercícios independentes em consola, cada um num pacote separado. Explora conceitos avançados de Kotlin como sealed classes, generics, lambdas, DSL e operator overloading.

### CoolWeatherApp
Aplicação Android nativa em Kotlin que consulta a API Open-Meteo para obter dados meteorológicos em tempo real. O utilizador pode usar a localização GPS ou introduzir coordenadas manualmente. A app apresenta temperatura, vento, pressão e um ícone dinâmico consoante as condições e a hora do dia (dia/noite). Suporta múltiplos layouts (portrait, landscape, tablet).

### ImageGalleryApp
Aplicação Android que obtém imagens aleatórias da Dog CEO API e as apresenta numa lista (RecyclerView). Inclui botão de refresh, indicador de carregamento e tratamento de erros. Desenvolvida com o AntiGravity IDE seguindo uma abordagem de prompting incremental.

## 3. Architecture and Design

### TP2
```
TP2/
└── src/main/kotlin/
    ├── Main.kt
    └── cm/
        ├── exer1/exer1.kt   (Sealed classes + extension functions)
        ├── exer2/exer2.kt   (Generics - Cache<K,V>)
        ├── exer3/exer3.kt   (Pipeline DSL com lambda receivers)
        └── exer4/exer4.kt   (Vec2 - operator overloading + Comparable)
```

### CoolWeatherApp — Padrão MVVM
```
app/src/main/
├── AndroidManifest.xml
└── java/com/example/coolweatherapp/
    ├── MainActivity.kt       (View)
    ├── WeatherViewModel.kt   (ViewModel - LiveData + Thread)
    └── WeatherData.kt        (Model - data classes + enum WMO_WeatherCode)

res/
├── drawable/             (34 ícones meteorológicos XML)
├── layout/               (portrait)
├── layout-land/          (landscape telemóvel)
├── layout-sw600dp/       (tablet portrait)
├── layout-sw600dp-land/  (tablet landscape)
├── values/               (strings, colors, themes, arrays)
└── values-pt-rPT/        (strings em português)
```

### ImageGalleryApp — Padrão MVVM + Repository
```
app/src/main/
├── AndroidManifest.xml
└── java/com/dam15048/imagegalleryappfinal/
    ├── MainActivity.kt
    ├── adapter/ImageAdapter.kt
    ├── api/DogApiService.kt
    ├── model/
    │   ├── DogApiResponse.kt
    │   └── ImageItem.kt
    ├── repository/ImageRepository.kt
    └── viewmodel/ImageViewModel.kt
```

Decisões de design: separação de responsabilidades em camadas distintas (API → Repository → ViewModel → View), uso de LiveData para reatividade, Retrofit para chamadas HTTP e Glide para carregamento de imagens.

## 4. Implementation

### TP2

**Exercício 1 — Sealed Classes e Extension Functions**

Sistema de eventos de utilizador (login, compra, logout) com `sealed class`. Extension functions sobre `List<Event>` para filtragem por utilizador, cálculo de total gasto e processamento com lambdas.

```kotlin
sealed class Event {
    data class Login(val username: String, val timestamp: Long) : Event()
    data class Purchase(val username: String, val amount: Double, val timestamp: Long) : Event()
    data class Logout(val username: String, val timestamp: Long) : Event()
}

fun List<Event>.totalSpent(username: String): Double =
    filterByUser(username).filterIsInstance<Event.Purchase>().sumOf { it.amount }
```

**Exercício 2 — Generics: Cache\<K, V\>**

Cache genérica com operações `put`, `get`, `evict`, `size`, `getOrPut`, `transform`, `snapshot` e `filterValues`. O tipo genérico usa `K : Any, V : Any` para evitar valores nulos.

```kotlin
class Cache<K : Any, V : Any> {
    private val data = mutableMapOf<K, V>()
    fun getOrPut(key: K, default: () -> V): V { ... }
    fun transform(key: K, action: (V) -> V): Boolean { ... }
}
```

**Exercício 3 — Pipeline com DSL (Lambda Receiver)**

Classe `Pipeline` que encadeia transformações sobre listas de strings. O builder `buildPipeline { }` usa lambda receiver (`Pipeline.() -> Unit`) para configuração declarativa das etapas.

```kotlin
val pipeline = buildPipeline {
    adicionarEtapa("Trim") { linhas -> linhas.map { it.trim() } }
    adicionarEtapa("Filter errors") { linhas -> linhas.filter { it.contains("ERROR") } }
    adicionarEtapa("Uppercase") { linhas -> linhas.map { it.uppercase() } }
    adicionarEtapa("Add index") { linhas -> linhas.mapIndexed { i, l -> "${i+1}. $l" } }
}
```

**Exercício 4 — Operator Overloading: Vec2**

Vetor 2D com sobrecarga de operadores (`+`, `-`, `*`, unário `-`), `Comparable<Vec2>` por magnitude, produto escalar (`dot`) e normalização. Extension function `Double.times(vec)` para suportar `2.0 * vec`.

```kotlin
data class Vec2(val x: Double, val y: Double) : Comparable<Vec2> {
    operator fun plus(other: Vec2) = Vec2(x + other.x, y + other.y)
    fun magnitude() = sqrt(x * x + y * y)
    override fun compareTo(other: Vec2) = magnitude().compareTo(other.magnitude())
}
```

---

### CoolWeatherApp

**WeatherData.kt** — Data classes que mapeiam a resposta JSON da API Open-Meteo (`WeatherData`, `CurrentWeather`, `Hourly`). O enum `WMO_WeatherCode` mapeia os códigos meteorológicos WMO para nomes de drawables, com variantes dia/noite.

**WeatherViewModel.kt** — Executa a chamada HTTP numa `Thread` separada para não bloquear a UI. Usa Gson para deserializar o JSON e expõe os dados via `MutableLiveData<WeatherData>`.

```kotlin
fun fetchWeatherData(lat: Float, long: Float) {
    Thread {
        val weather = weatherApiCall(lat, long)
        weatherData.postValue(weather)
    }.start()
}
```

**MainActivity.kt** — Solicita permissão de localização em runtime, obtém as coordenadas GPS, observa o LiveData e atualiza a UI com o tema correto (dia/noite) e o layout adequado à orientação e tamanho do ecrã.

Permissões: `INTERNET`, `ACCESS_FINE_LOCATION`, `ACCESS_COARSE_LOCATION`.
Dependências: `Gson 2.10.1`, `lifecycle-viewmodel-ktx 2.8.4`, `lifecycle-livedata-ktx 2.8.4`.
API: `https://api.open-meteo.com/v1/forecast`

---

### ImageGalleryApp

**DogApiService.kt** — Interface Retrofit com endpoint `GET breeds/image/random/10`.

**ImageRepository.kt** — Configura o Retrofit com base URL `https://dog.ceo/api/` e converte a resposta para `List<ImageItem>` com IDs UUID.

**ImageViewModel.kt** — Usa `viewModelScope.launch` (coroutines) para chamadas assíncronas, expondo `images`, `loading` e `error` como LiveData.

```kotlin
fun loadImages() {
    viewModelScope.launch {
        _loading.value = true
        try {
            _images.value = repository.getImages()
        } catch (e: Exception) {
            _error.value = "Failed to load images: ${e.message}"
        } finally {
            _loading.value = false
        }
    }
}
```

**ImageAdapter.kt** — `RecyclerView.Adapter` com Glide para carregamento de imagens com `centerCrop`.

Dependências: Retrofit, Gson Converter, Glide, ViewModel/LiveData KTX, RecyclerView.
Permissões: `INTERNET`.

## 5. Testing and Validation

**TP2:** Executado em IntelliJ IDEA com verificação da saída de consola. Casos de borda testados: cache com chave inexistente (retorna `null`), vetor zero em `normalized()` (lança `IllegalStateException`), utilizador sem eventos (lista vazia e total 0.0).

**CoolWeatherApp:** Testado em AVD Pixel 9 Pro (API 34) e dispositivo físico. Verificada a resposta ao botão de atualização, alternância de temas dia/noite e layouts portrait/landscape em telemóvel e tablet.

**ImageGalleryApp:** Testado em emulador Android. Verificado o carregamento inicial, botão de refresh, visibilidade do ProgressBar e mensagem de erro em caso de falha de rede.

Limitações conhecidas: CoolWeatherApp usa `Thread` direta em vez de coroutines; ImageAdapter usa `notifyDataSetChanged()` em vez de `DiffUtil`.

## 6. Usage Instructions

### TP2
1. Abrir o projeto em IntelliJ IDEA (Maven).
2. Executar cada ficheiro (`exer1.kt` a `exer4.kt`) individualmente como `main()`.

### CoolWeatherApp
1. Abrir o projeto em Android Studio.
2. Sincronizar o Gradle.
3. Executar num AVD (Pixel 9 Pro, API 34) ou dispositivo físico com Android 7.0+.
4. Conceder permissão de localização ou introduzir coordenadas manualmente e pressionar "Update".

### ImageGalleryApp
1. Abrir o projeto em Android Studio (ou AntiGravity IDE).
2. Sincronizar o Gradle.
3. Executar num emulador ou dispositivo físico com Android 7.0+.
4. As imagens carregam automaticamente. Pressionar refresh para recarregar.

---

# Autonomous Software Engineering Sections - only for [AC OK, AI OK] sections

## 7. Prompting Strategy

Para o **ImageGalleryApp**, foi adotada uma estratégia de prompting incremental usando o AntiGravity IDE em Planning Mode. Cada prompt focava uma única responsabilidade:

| Etapa | Prompt utilizado |
|-------|-----------------|
| Dependências e permissões | Adicionar Retrofit, Gson, Glide, ViewModel, LiveData, RecyclerView ao Gradle; adicionar `INTERNET` ao Manifest |
| Modelos de dados | Criar `DogApiResponse` e `ImageItem` |
| API Service | Criar `DogApiService` com Retrofit |
| Repository | Criar `ImageRepository` |
| ViewModel | Criar `ImageViewModel` |
| Layouts | Criar `activity_main.xml` e `item_image.xml` com XML Views |
| Adapter | Criar `ImageAdapter` para RecyclerView |
| MainActivity | Ligar ViewModel, RecyclerView, adapter, botão, ProgressBar e TextView de erro |
| Verificação | Build do projeto e validação no emulador |

O ficheiro `agents.md` definiu regras persistentes: Kotlin only, XML Views, sem Jetpack Compose, abordagem step-by-step seguindo o plano em `/docs`.

## 8. Autonomous Agent Workflow

1. Definição das especificações em ficheiros Markdown (`/docs/*.md`).
2. Aprovação do plano de implementação gerado pelo agente (`docs/08_implementation_plan.md`).
3. Autorização das comandas de terminal propostas pelo agente.
4. Validação incremental após cada etapa (build + execução no emulador).
5. Iteração com o agente para correção de eventuais erros de compilação.

O agente contribuiu para: estrutura do projeto, configuração do Gradle, implementação de todas as camadas (API, Repository, ViewModel, Adapter, Activity) e resolução de erros de build.

## 9. Verification of AI-Generated Artifacts

- Revisão manual de cada ficheiro gerado antes de aceitar as alterações.
- Build do projeto com verificação de erros de compilação.
- Execução no emulador para validação funcional (carregamento de imagens, refresh, loading, erros).
- Leitura e compreensão do código gerado, incluindo o funcionamento de Retrofit, coroutines com `viewModelScope` e Glide.

## 10. Human vs AI Contribution

| Componente | Contribuição |
|---|---|
| TP2 — Exercícios Kotlin (exer1 a exer4) | Humano (AI e AC proibidos) |
| CoolWeatherApp — Toda a implementação | Humano (AI proibido nesta secção) |
| ImageGalleryApp — Especificações `/docs` e `agents.md` | Humano |
| ImageGalleryApp — Código Kotlin e layouts | IA (AntiGravity IDE) |
| ImageGalleryApp — Revisão, testes e validação | Humano |

## 11. Ethical and Responsible Use

O uso de IA foi restrito às secções explicitamente autorizadas (`[AC YES, AI YES]`). Nas secções proibidas (`[AC NO, AI NO]` e `[AC YES, AI NO]`), nenhuma ferramenta de geração de código foi utilizada.

Todo o código gerado pela IA foi revisto e compreendido antes de ser aceite. O aluno mantém total responsabilidade pelo conteúdo submetido. Risco identificado: dependência excessiva em IA pode reduzir a capacidade de resolução independente — mitigado pela realização dos exercícios TP2 sem qualquer auxílio.

---

# Development Process

## 12. Version Control and Commit History

O desenvolvimento foi realizado sob controlo de versão Git. O histórico de commits reflete a progressão incremental do trabalho, desde a inicialização da estrutura até à implementação final de cada componente. Os commits foram realizados após cada etapa significativa.

## 13. Difficulties and Lessons Learned

- **Permissões em runtime**: a solicitação de permissões de localização requer tratamento de callbacks (`onRequestPermissionsResult`), diferente do desenvolvimento desktop.
- **Threading no Android**: compreender a diferença entre usar `Thread` direta (CoolWeatherApp) e coroutines com `viewModelScope` (ImageGalleryApp).
- **MVVM e LiveData**: perceber o padrão observer e como a UI reage automaticamente a mudanças de estado sem acoplamento direto.
- **Layouts responsivos**: criar variantes de layout para orientações e tamanhos de ecrã diferentes exige planeamento cuidado das constraints.
- **Prompting incremental**: prompts focados numa única responsabilidade produzem código mais limpo e fácil de validar.

## 14. Future Improvements

- **CoolWeatherApp**: migrar `Thread` para coroutines; adicionar previsão horária em gráfico; pesquisa por nome de cidade; suporte a unidades imperiais.
- **ImageGalleryApp**: substituir `notifyDataSetChanged()` por `DiffUtil`; suporte a múltiplas APIs; favoritos com Room; grid layout alternativo.
- **TP2**: adicionar testes unitários com JUnit para cada exercício.

---

## 15. AI Usage Disclosure (Mandatory)

| Ferramenta | Utilização | Secção |
|---|---|---|
| AntiGravity IDE | Geração de código Kotlin para o ImageGalleryApp (modelos, API, repository, ViewModel, adapter, activity, layouts) | MIP [AC YES, AI YES] |

O aluno confirma que reviu todo o código gerado pela IA, compreende o seu funcionamento e assume total responsabilidade pelo conteúdo submetido. Nenhuma ferramenta de IA foi utilizada nas secções marcadas como `[AC NO, AI NO]` ou `[AC YES, AI NO]`.
