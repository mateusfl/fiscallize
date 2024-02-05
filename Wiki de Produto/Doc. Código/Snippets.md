Snippets de código são pequenos trechos de programa que podem ser reutilizados para resolver problemas pontuais. Esses trechos são especialmente úteis para programadores que precisam de soluções rápidas e eficientes para problemas comuns. Nesta página, você encontrará uma seleção de snippets que podem ser utilizados em diversas situações.

  

## Ajustes de Students duplicados

O problema ocorrido no colégio Meta recai na duplicidade dos alunos (Student) e com isso os application_students estão inconsistentes. A resolução se dá em dois passos:

1. O time de CS identifica os alunos com problema e adiciona “ERRADO” ao final do nome do `Student`.
2. Os `ApplicationStudent` relacionados a esse aluno são atualizados para o `Student` correto:

```Python
year = 2023
client_id = 'CLIENT_PK'
incorrect_students = students = Student.objects.filter(
    email__icontains='@email-temp.com.br',
    client=client_id
) 
multiple_students = []
unique_students = []
for student in incorrect_students:
    application_students = ApplicationStudent.objects.filter(
        student=student, 
        created_at__year=year
    )
    for application_student in application_students:
        correct_student = Student.objects.filter(
            client=client_id, 
            name__icontains=student.name.split('@')[0].strip()
        ).exclude(
            email__icontains='@email-temp.com.br'
        )
        if correct_student.count() > 1:
            multiple_students.append(student.pk)
            break
        elif correct_student.count() == 0:
            unique_students.append(student.pk)
            break
        print(f'ApplicationStudent: {application_student.pk} | StudentErrado: {student.pk} | StudentCorreto: {correct_student[0].pk}')
        application_student.student = correct_student.first()
        application_student.save()
```