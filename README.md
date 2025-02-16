# 📌 Jiu-Jitsu School REST API

## 📝 Sobre o Projeto
Esta é uma aplicação REST API desenvolvida com **Python** e **Django REST Framework**, projetada para permitir que escolas de Jiu-Jitsu acompanhem o progresso de seus alunos de forma eficiente e organizada.

## 🚀 Funcionalidades
- Cadastro de alunos
- Registro de aulas concluídas
- Cálculo automático da progressão de faixas
- Atualização de informações dos alunos

## 📦 Tecnologias Utilizadas
- **Python**
- **Django**
- **Django REST Framework**
- **NinjaAPI**
- **Banco de dados relacional**

---

## 📌 Como Rodar o Projeto

### 🔹 1. Criar e Ativar o Ambiente Virtual
#### Linux:
bash
python3 -m venv venv
source venv/bin/activate

#### Windows:
bash
python -m venv venv
venv\Scripts\Activate


Caso haja erro de permissão, execute:
powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned


### 🔹 2. Instalar Dependências
bash
pip install django pillow django-ninja


### 🔹 3. Criar o Projeto Django
bash
django-admin startproject core .
python manage.py runserver


### 🔹 4. Criar o App
bash
python manage.py startapp treino


### 🔹 5. Configurar as URLs da API
No arquivo **urls.py**, adicione:
python
from .api import api
path('api/', api.urls)


No arquivo **core/api.py**, registre o roteador:
python
from ninja import NinjaAPI
from treino.api import treino_router

api = NinjaAPI()
api.add_router('', treino_router)


---

## 📌 Modelos e Endpoints

### 📍 Modelo de Alunos
python
class Alunos(models.Model):
    faixa_choices = (
        ('B', 'Branca'),
        ('A', 'Azul'),
        ('R', 'Roxa'),
        ('M', 'Marrom'),
        ('P', 'Preta')
    )
    nome = models.CharField(max_length=255)
    email = models.EmailField(unique=True)
    faixa = models.CharField(max_length=1, choices=faixa_choices, default='B')
    data_nascimento = models.DateField(null=True, blank=True)


### 📍 Schema de Alunos
python
from ninja import ModelSchema
from .models import Alunos

class AlunosSchema(ModelSchema):
    class Meta:
        model = Alunos
        fields = ['nome', 'email', 'faixa', 'data_nascimento']


### 📍 Criar um Aluno
python
@treino_router.post('/', response={200: AlunosSchema})
def criar_aluno(request, aluno_schema: AlunosSchema):
    if Alunos.objects.filter(email=aluno_schema.email).exists():
        raise HttpError(400, "E-mail já cadastrado.")
    aluno = Alunos(**aluno_schema.dict())
    aluno.save()
    return aluno


### 📍 Listar Alunos
python
@treino_router.get('/alunos/', response=List[AlunosSchema])
def listar_alunos(request):
    return Alunos.objects.all()


### 📍 Modelo de Aulas Concluídas
python
class AulasConcluidas(models.Model):
    aluno = models.ForeignKey(Alunos, on_delete=models.CASCADE)
    data = models.DateField(auto_now_add=True)
    faixa_atual = models.CharField(max_length=1, choices=faixa_choices)


### 📍 Cálculo de Progressão de Faixa
python
def calculate_lessons_to_upgrade(n):
    import math
    d = 1.47
    k = 30 / math.log(d)
    return round(k * math.log(n + d))


### 📍 Ver Progresso do Aluno
python
@treino_router.get('/progresso_aluno/', response={200: ProgressoAlunoSchema})
def progresso_aluno(request, email_aluno: str):
    aluno = Alunos.objects.get(email=email_aluno)
    total_aulas_concluidas = AulasConcluidas.objects.filter(aluno=aluno).count()
    faixa_atual = aluno.get_faixa_display()
    n = order_belt.get(faixa_atual, 0)
    total_aulas_proxima_faixa = calculate_lessons_to_upgrade(n)
    return {"nome": aluno.nome, "email": aluno.email, "faixa": faixa_atual, "total_aulas": total_aulas_concluidas, "aulas_necessarias_para_proxima_faixa": total_aulas_proxima_faixa}


### 📍 Atualizar Dados do Aluno
python
@treino_router.put("/alunos/{aluno_id}", response=AlunosSchema)
def update_aluno(request, aluno_id: int, aluno_data: AlunosSchema):
    aluno = get_object_or_404(Alunos, id=aluno_id)
    for attr, value in aluno_data.dict().items():
        if value:
            setattr(aluno, attr, value)
    aluno.save()
    return aluno


---

## 📌 Como Contribuir
1. Faça um **fork** do repositório.
2. Crie uma nova **branch**: `git checkout -b minha-feature`
3. Commit suas mudanças: `git commit -m 'Adiciona nova feature'`
4. Faça um push da branch: `git push origin minha-feature`
5. Abra um **Pull Request**.

---


💡 **Desenvolvido com Django Ninja para facilitar a criação de APIs escaláveis!** 🚀

