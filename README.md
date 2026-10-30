# Mvga: Coordenadas Baricêntricas com OpenGL

Programa em **C++ / OpenGL 3.3 (core profile)** que desenha um triângulo ABC em um *fragment shader* de tela cheia, calcula as **coordenadas baricêntricas** de um ponto em função dos vértices e **muda a coloração do triângulo conforme a posição do mouse** (dentro ou fora).

---

## Sumário

1. [Como funciona](#como-funciona)
2. [Requisitos](#requisitos)
3. [Estrutura do repositório](#estrutura-do-repositório)
4. [Configuração no Windows](#configuração-no-windows)
5. [Execução e controles](#execução-e-controles)
6. [Uniforms disponíveis nos shaders](#uniforms-disponíveis-nos-shaders)
7. [Linux (adaptação necessária)](#linux-adaptação-necessária)
8. [Problemas comuns](#problemas-comuns)
9. [Créditos](#créditos)


---

## Como funciona

1. `Barycentric.cpp` cria a janela (GLFW), carrega as funções do OpenGL (GLAD) e compila os shaders.
2. Um quadrado de tela cheia (dois triângulos) é desenhado a cada frame; o *fragment shader* `Shaders/Image.fs` roda **uma vez por pixel**.
3. No shader, dado um ponto P e um triângulo ABC, escrevemos

   `P = α·A + β·B + γ·C`, com `α + β + γ = 1`

   e o ponto está **dentro** do triângulo quando `α > 0`, `β > 0` e `γ > 0`.
4. O C++ envia ao shader a resolução, o tempo e a posição do mouse; o shader decide a cor de cada pixel.

> O fragment shader não imprime texto. Mensagens como "dentro/fora" devem ser exibidas no lado C++ (terminal ou título da janela).

---

## Requisitos

| Item | Observação |
|---|---|
| Compilador C++17 | MinGW-w64 (g++) ou MSVC (Visual Studio 2019+) |
| [CMake](https://cmake.org/download/) ≥ 3.16 | Usado para gerar o build |
| Placa/driver com **OpenGL 3.3** ou superior | Atualize o driver de vídeo se a janela não abrir |
| [GLFW](https://www.glfw.org/download.html) (binários pré-compilados) | Biblioteca **fora do repositório**; precisa ser baixada |
| GLAD | `glad.c` está na raiz; os cabeçalhos ficam em `includes/` |
| GLM | Matemática de vetores/matrizes (cabeçalhos em `includes/`) |
| stb_image | Já incluso em `stb_image/` e `stb_image.cpp` |

---

## Estrutura do repositório

```
Mvga/
├── Barycentric.cpp        # programa principal (janela, laço de render, entrada do mouse)
├── Shader.h               # classe Shader (lê, compila e linka os shaders)
├── glad.c                 # carregador de funções OpenGL
├── stb_image.cpp          # implementação do stb_image
├── CMakeLists.txt         # script de build
├── configuration/         # root_directory.h.in (gera o root_directory.h no build)
├── includes/              # cabeçalhos de terceiros (glad, GLM, learnopengl/filesystem.h, ...)
├── stb_image/             # cabeçalho do stb_image
├── Shaders/               # shaders GLSL (copiados para a pasta do executável)
│   ├── VertexShader.vs
│   ├── Image.fs           # fragment shader do triângulo / baricêntricas
│   └── common.incl        # trecho comum, colocado antes de cada fragment shader
└── malha.off              # malha de exemplo
```

Dois detalhes importantes sobre os shaders:

- `Shader.h` **concatena** `common.incl` antes do código do fragment shader. É em `common.incl` que ficam as declarações comuns aos shaders.
- Os caminhos dos shaders são montados a partir do **diretório atual de execução** (`/Shaders/...`). Por isso o programa deve ser executado a partir da pasta onde está o `Shaders/` copiado (o CMake já faz essa cópia para a pasta do executável).

---

## Configuração no Windows

O `CMakeLists.txt` atual é voltado ao Windows (liga `glfw3dll`, `opengl32` e `gdi32`).

### 1. Obter o GLFW

1. Baixe os **pré-compilados para Windows (64-bit)** em <https://www.glfw.org/download.html>.
2. Extraia em uma pasta de sua escolha. Dentro dela devem existir:
   - `include/GLFW/glfw3.h`
   - `lib-mingw-w64/` **ou** `lib-vc20xx/` (use a que corresponde ao seu compilador), contendo `glfw3dll.lib` / `libglfw3dll.a` e `glfw3.dll`

### 2. Ajustar o caminho do GLFW no `CMakeLists.txt`

A linha abaixo está com o caminho da máquina do autor e **precisa ser alterada**:

```cmake
set(GLFW_DIR "C:/Projects/Studies_in_C/Exercises/openGL")
```

Troque pela pasta onde você extraiu o GLFW, e copie ou aponte os arquivos de `lib-*` para uma pasta `lib/`, já que o script usa `${GLFW_DIR}/include` e `${GLFW_DIR}/lib`:

```
<GLFW_DIR>/
├── include/GLFW/glfw3.h
└── lib/glfw3dll.lib   (ou libglfw3dll.a no MinGW)
```

### 3. Gerar e compilar

**MinGW-w64 (g++):**

```powershell
cmake -S . -B build -G "MinGW Makefiles"
cmake --build build
```

**Visual Studio (MSVC):**

```powershell
cmake -S . -B build -G "Visual Studio 17 2022"
cmake --build build --config Release
```

### 4. Colocar a DLL do GLFW ao lado do executável

Copie `glfw3.dll` para a mesma pasta de `Barycentric.exe`:

- MinGW: `build\`
- MSVC: `build\Release\`

Sem isso, o Windows exibe o erro de DLL não encontrada ao abrir o programa.

---

## Execução e controles

Execute a partir da pasta do executável (onde está `Shaders/`):

```powershell
cd build            # ou build\Release no Visual Studio
.\Barycentric.exe
```

> **Atenção:** o programa inicia **pausado**. A janela fica preta até você pressionar **Espaço**.

| Entrada | Ação |
|---|---|
| **Espaço** | Liga/desliga o desenho (a tela fica preta enquanto estiver desligado) |
| **Botão esquerdo do mouse** | Informa ao shader que o mouse está pressionado (`iMouse.z = 1`) |
| **Botão direito do mouse** | `iMouse.w = 1` |
| **Esc** | Fecha o programa |
| **I** | Alterna a variável `inflow` (usada por shaders de buffer, hoje desativados) |

Após ligar o desenho, o título da janela mostra o FPS.

---

---

## Linux (adaptação necessária)

O `CMakeLists.txt` atual **não compila no Linux** sem alterações (as bibliotecas `glfw3dll`, `opengl32` e `gdi32` são do Windows). Uma adaptação possível, **não testada**:

```bash
sudo apt-get install g++ cmake git libglfw3-dev libglm-dev libgl1-mesa-dev xorg-dev
```

No `CMakeLists.txt`, substitua a configuração do GLFW e a linha `target_link_libraries` por:

```cmake
if(WIN32)
    set(GLFW_DIR "C:/caminho/para/glfw")
    include_directories(${GLFW_DIR}/include)
    link_directories(${GLFW_DIR}/lib)
    target_link_libraries(Barycentric glfw3dll opengl32 gdi32)
else()
    find_package(glfw3 REQUIRED)
    target_link_libraries(Barycentric glfw GL dl)
endif()
```

Como `target_link_libraries` deve vir depois de `add_executable`, mantenha esse bloco abaixo dela.

---

## Problemas comuns

| Sintoma | Causa provável | Solução |
|---|---|---|
| Janela preta | O desenho começa desligado | Pressione **Espaço** |
| `ERROR::SHADER::FILE_NOT_SUCCESFULLY_READ` | Programa executado fora da pasta que contém `Shaders/` | Execute a partir da pasta do executável |
| `ERROR::SHADER_COMPILATION_ERROR` | Erro de sintaxe GLSL | Leia o log impresso no terminal (indica a linha) |
| `Failed to create GLFW window` | Driver sem suporte a OpenGL 3.3 | Atualize o driver de vídeo |
| `Failed to initialize GLAD` | Contexto OpenGL não criado | Verifique driver e se `glad.c` está no `add_executable` |
| `GLFW/glfw3.h: No such file` ou `cannot find -lglfw3dll` | `GLFW_DIR` incorreto | Confira o caminho no `CMakeLists.txt` e a estrutura `include/` e `lib/` |
| `learnopengl/filesystem.h: No such file` | Pasta `includes/` incompleta | Garanta que `includes/learnopengl/filesystem.h` existe e que o CMake rodou (gera `root_directory.h`) |
| Erro de DLL ao abrir o `.exe` | `glfw3.dll` ausente | Copie `glfw3.dll` para a pasta do executável |
| `Texture failed to load at path: ...Lapis.png` | A textura de exemplo não está em `Shaders/` | Mensagem informativa; não afeta o triângulo |

---

## Créditos

- A estrutura do projeto (classe `Shader`, `loadTexture`, `learnopengl/filesystem.h`) é inspirada no material do **[LearnOpenGL](https://learnopengl.com)**, de Joey de Vries.
- [GLFW](https://www.glfw.org), [GLAD](https://glad.dav1d.de), [GLM](https://github.com/g-truc/glm) e [stb_image](https://github.com/nothings/stb).

Autor: [Matosaz](https://github.com/Matosaz)
