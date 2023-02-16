<img width="1584" alt="W0-B" src="https://user-images.githubusercontent.com/123767474/219365375-7a054844-6812-4ef6-ae0d-2657faf25691.png">

# Week 0 — Billing and Architecture

![GitHub last commit](https://img.shields.io/github/last-commit/ash-codess/aws-bootcamp-cruddur-2023)

## Introduction

The bootcamp has officially kicked off, and I couldn't be more thrilled about the journey ahead.
The instructors were not only incredibly knowledgeable, but also really nice, which set a positive and collaborative tone for the entire program. I'm excited to learn more and work through 12 weeks of the bootcamp program!

<details>
<summary> Youtube live takeaway </summary>

Few things i picked from livestream that i wanted to look into more:

- **Iron triangle** - The basic idea is that any project has a fixed amount of time, a fixed budget or cost, and a specific set of deliverables or scope that must be met. Changing any one of these factors will affect the other two. For example, if the scope of a project is increased, then either the cost or the time required to complete the project will have to increase as well. Similarly, if the time available for a project is reduced, then either the scope or the cost will have to be reduced as well.

  ![iron-triangle](https://user-images.githubusercontent.com/123767474/219365702-a4a7dcde-e648-43e9-9204-062102a56336.png)

- **TOGAF** - We need TOGAF or similar enterprise architecture frameworks to provide a structured and organized approach to managing the complexity of large IT systems and to align them with the organization's business goals. By using a standardized approach, it becomes easier to communicate and collaborate between different teams and departments.(Didn't look more into as instructed by Chris, lol)

- **Adrain Cantril’s CI/CD pipeline mini project** - The goal of CI/CD is to enable faster and more reliable software delivery by reducing the time and effort required to move code changes from development to production. I followed Adrian's mini project and implemented an event-driven pipeline, it was a video-on demand backend service which will take a video uploaded on s3 and with the help of aws media-converter it will convert it into different formats like (HD/SD) and more!

- **AWS well-architected framework** - I checked out the AWS well-architected tool and tried to fill out the questions for the cruddur. The set of questions that were in it was quiet vast. I plan to link the generated report down below as a part of extended homework.
</details>

---

<details>
<summary>Required Homework</summary>
<br>

- Recreate Conceptual Diagram in Lucid Charts or on a Napkin
  ![Napkin diagram](https://user-images.githubusercontent.com/123767474/219365837-cecf0682-f289-44f8-b2a7-eefafa144fcb.jpg)
- **Recreate Logical Architectural Diagram in Lucid Charts**
  ![Logical Architectural Diagram](https://user-images.githubusercontent.com/123767474/219365946-6a36f12d-b992-42d9-a1fc-40816325cba5.png)
  [Lucid chart link](https://lucid.app/lucidchart/59e7df73-0879-4b64-9694-bfe3e89effed/edit?viewport_loc=-298%2C-228%2C3328%2C1642%2C0_0&invitationId=inv_dbc91856-3cb0-43e3-b7cc-e811ec27b9c1)

- I followed the week 0 instruction and was able to successfully do the setup. For journal i am using vscode, as it is easier to see the changes i make simultaneously and do one final push once i am satisfied with the work. I have a clone of repo in my local system.
  ![Vs-code proof](https://user-images.githubusercontent.com/123767474/219366049-9e776535-fc10-4043-8a96-b373f841fc4d.png)

- Few proof of work i would like to show - Destroyed my root account credential and everything is done admin IAM user

- **Budget**

  ![Vs-code proof](https://user-images.githubusercontent.com/123767474/219366154-e17f1733-bc63-43c4-bf77-44508b7a4680.png)

- **Billing alarm**
  <br />
  So I ran into my first issue when i checked SNS to take screenshot i saw there was nothing there the notification that i created was gone. I thought maybe it was accidentally created into my root account so i cross-checked but that wasn't the case.Turns out i made AWS send me couple of notification conformation mails because it landed in spam folder and i didn't click on the right link for conformation and sns deletes the topic after three days of pending conformation.
  I performed this step again to create an alarm.
  ![W0-5](https://user-images.githubusercontent.com/123767474/219381037-bea40cf9-e41d-4bef-ae98-9d8330091ff4.png)


</details>

---

<details>
<summary> Extended Homework </summary>

Pending

</details>
