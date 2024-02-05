  

## Entendendo como funciona

  

No Django, as permissões são definidas em nível de modelo e podem ser atribuídas a grupos ou usuários individuais. Existem três tipos principais de permissões: "add" (adicionar), "change" (alterar) e "delete" (excluir). Essas permissões controlam o que os usuários podem fazer com os objetos do modelo, como criar, editar ou excluir registros.

Além das permissões padrão, o Django também permite que você defina permissões personalizadas para atender às necessidades específicas da sua aplicação. Isso oferece flexibilidade para controlar o acesso dos usuários a diferentes partes da aplicação.

Ao usar o sistema de permissões do Django, é importante garantir que as permissões sejam aplicadas corretamente em todas as views e templates relevantes. O Django oferece uma variedade de mecanismos para verificar as permissões dos usuários, como decorators e tags de template, que facilitam a implementação e o controle do acesso com base nas permissões atribuídas.

Em resumo, o sistema de permissões do Django é uma ferramenta poderosa para controlar o acesso dos usuários em aplicações web. Com ele, você pode definir permissões de forma granular e garantir que apenas os usuários autorizados possam realizar determinadas ações em sua aplicação.

  

No modelo `User` temos as configurações de permissões

![[Untitled 2.png|Untitled 2.png]]

  

Quando criamos um modelo novo o django adiciona algumas permissões como foi descrito acima.

as permissões são: `view_` `add_` `change_` `delete_`

Existe um padrão na criação, e o padrão é a inicial da permissão + o nome do modelo em minusculo

  

Exemplo para o modelo Exam:

as permissões são:

`view_exam`

`add_exam`

`change_exam`

`delete_exam`

  

Além das permissões padrões você pode criar permissões personalizadas como no exemplo abaixo:

No `Meta` do modelo `Exam` eu criei a seguinte permissões personalizada:

![[Untitled 1.png]]

  

  

## Como utilizar as permissões nas views

  

Em todas as views que você precisar checar as permissões você precisa adicionar um mixin do Django chamado `PermissionRequiredMixin` que é de `from django.contrib.auth.mixins import PermissionRequiredMixin`

  

Esse Mixin é responsável por fazer a validação das permissões dos usuários.

  

Para que ele funcione você precisa implementar a variável de permissões que é a `permission_required` essa variável pode ser uma string com uma única permissão ou um array com várias permissões.

  

Exemplo da **view** de **Exam**:

quero checar se na minha listagem de cadernos o usuário tem a permissão padrão para Exam que é a permissão `view_exam`

  

Eu vou adicionar a minha view a seguinte permissão:

`permission_required = ['exams.view_exam']`

  

Com isso eu garanto que apenas usuários com essa permissão irão acessar a listagem de cadernos.

  

Para montar a checagem de permissões nessa variável `permission_required` você precisa especificar qual app você quer pegar a permissão, isso evita duplicidade de permissões.

  

  

## Como utilizar as permissões nos templates

  

O modelo User estende uma classe chamada `PermissionsMixin` e é dessa classe que o django gerencia as permissões dos usuários.

  

Essa classe é responsável por gerenciar e controlar as permissões dos usuários, e dentro dela tem alguns métodos importantes, um deles é o `get_all_permissions`

  

Caso você precise esconder um informação do frontend caso o usuário não tenha determinada permissão você pode utilizar esse método para checar as permissões do usuário, caso ele não tenha determinada permissão você oculta a informação.

  

**Exemplo para ocultar uma informação do usuário no frontend**

`{% if 'exams.delete_exam' not in user.get_all_permissions %} <button>REMOVER CADERNO</button> {% endif %}`

  

existe também um método chamado `has_perm` que pode ser utilizado com ajuda de um templatetag, dentro de `core.templatetags.permission` existe um templatetag chamado `has_perm`

  

você pode utiliza-lo se preferir, nesse caso o IF mudaria um pouco como no exemplo abaixo:

`{% if not user|has_perm:'exams.delete_exam' %} <button>REMOVER CADERNO</button> {% endif %}`

  

A implementação fica a seu critério.