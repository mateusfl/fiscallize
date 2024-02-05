  

Existem algumas situações onde é preciso distribuir os pesos de uma questão ou do caderno inteiro.

  

Para lidar com isso, temos a função `**distribute_weights**` de **`Exam`** que é chamada via hook com a lib django_lifecycle_hooks: [https://rsinger86.github.io/django-lifecycle/](https://rsinger86.github.io/django-lifecycle/) em algumas situações

  

Situações:

1. Quando o valor do campo **`subject_note`** de **`ExamTeacherSubject`** é alterado.
2. Quando o valor `**total_grade**` do `**Exam**` é alterado.
3. Quando o valor da variável total`**total_grade**` é maior que **zero** e alguma questão é **adicionada,** **removida, reprovada** ou **aprovada**.

  

[`**CTRL+SHIFT+F:**`](https://github.com/franklindias/fiscallize-online/blob/b3ecca1233fa63035223cbe3fc7608726d1c685c/fiscallizeon/exams/models.py#LL1093C9-L1093C32) [_`**DOC_KWTZH4EQ**`_](https://github.com/franklindias/fiscallize-online/blob/b3ecca1233fa63035223cbe3fc7608726d1c685c/fiscallizeon/exams/models.py#LL1093C9-L1093C32)

`class ExamTeacherSubject(BaseModel):     @hook("after_update", when="subject_note", has_changed=True)     def update_questions_weight(self):         if self.subject_note:             exam = self.exam             exam.distribute_weights(exam_teacher_subject=self)`

  

[`**CTRL+SHIFT+F:**`](https://github.com/franklindias/fiscallize-online/blob/b3ecca1233fa63035223cbe3fc7608726d1c685c/fiscallizeon/exams/models.py#LL168C4-L168C4) [`**_DOC_GTMPHNN6_**`](https://github.com/franklindias/fiscallize-online/blob/b3ecca1233fa63035223cbe3fc7608726d1c685c/fiscallizeon/exams/models.py#LL168C4-L168C4)

`class Exam(BaseModel):     @hook("after_update", when="total_grade", has_changed=True)     def distribute_weights(self, exam_teacher_subject=None):`