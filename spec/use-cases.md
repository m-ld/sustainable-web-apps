# use-cases

Our project narrative intentionally encompasses a wide range of end-user use-cases. We want to show that no compromise is necessary in building a '[sustainable](sustainability.md)' app versus building an unsustainable one, for a large set of them.

However, we are clearly not going to demonstrate radical improvements to every possible web app. So actually, our 'users' are the engineers who build those apps. We look at use-cases not only from the end-user perspective but also from the engineer perspective, to motivate our design hypothesis. In the context of this project, we especially want to consider cases that we can usefully demonstrate, taking the role of application developers ourselves; and still give confidence that the solution will scale to production features and workloads.

The following use-cases are primarily drawn from the personal experience of the project team, both as users and as engineers.

### terminology

In the following we make use of software engineering terms, intending their usual meaning. Here are some definitions of the more esoteric ones:

| term                      | meaning                                                                                                                  |
|---------------------------|--------------------------------------------------------------------------------------------------------------------------|
| partially structured data | data whose structure is at least partially described in a data model, which may be abstract with respect to some content |
| variety (big data axis)   | an abstract expression of the diversity of data types and relationships in data, and how frequently these can change     |
| veracity (big data axis)  | an abstract expression of the quality of data, including its precision and accuracy                                      |

## Project Management

"As a project manager, I want to facilitate real-time collaboration on project planning and tracking, enabling team members to update tasks, dependencies, and progress in real-time, so that everyone is on the same page."

### User Challenges

1. Over the course of a programme of projects, teams frequently want to change aspects of their tooling, including specifically the project management tools. They are inhibited from doing this by limited options to export or migrate their data, and by restrictive pricing models.
2. In attempting to fit all possible projects, project management apps can be overly complex and difficult to navigate, requiring a steep learning curve for users. This can hinder adoption and efficiency.
3. Project management involves **sensitive information** related to timelines, resources, and team collaboration. Existing apps may have privacy and data security concerns, whether the app is deployed locally or on the cloud.
4. While collaboration is at the heart of project management, apps often adopt a locking strategy for editing content. This can hinder free flow of ideas and information during a collaboration session; for example when a manager has to transcribe another team member's input.

### Engineering Challenges

1. Project management information is an example of partially structured data with significant requirements on the variety and veracity 'big data' axes:
   - Combination of strictly typed and controlled fields and forms, with free-form rich text
   - Business process management features such as workflow
   - Significant desire to tailor the above based on context and project needs
   - Links to other systems such as Customer Relationship Management, Customer Support, Source Code Control
   - Summarisation and dashboarding for reporting to stakeholders
2. Features tend to have been added ad-hoc during the product's lifecycle, resulting in significant technical debt.
3. Specifically, the need for **live collaborative editing** and **system integration APIs** tend to come later in the product's maturity cycle, once customers are already on-board. These are well-known to be hard to retrofit [1].

### Demonstration

A full project management application is clearly a significant engineering undertaking. However, its essence is to enable planning and tracking of a 'project'; and in many systems the information structure is reduced (fairly successfully) to individual data items known variously as 'tickets' or 'issues' or 'cards'.

It should be sufficient to demonstrate that an application structured like this can be built in such a way that the challenges above are addressed early on, without affecting the speed of delivery and the eventual feature richness.

## Note-Taking

"As a thought-worker, I want to take notes in the course of my research in a structured and managed way, so that I can efficiently drive to a justified conclusion that is ready to write up."

### User Challenges

1. Many note-taking apps have limitations when it comes to organizing and structuring notes. Thought-workers often need to categorise and tag their notes, create hierarchies, or link related information, including for formal citations.
2. Note-taking apps may not offer easy ways to export or migrate data, resulting in data lock-in.
3. Team collaboration is essential for many projects. Existing note-taking apps may lack robust collaboration features, making it challenging for thought-workers to collaborate efficiently, share information, and work together in real-time.
4. Intellectual work often involves sensitive and confidential information 'property'. Existing note-taking apps may have privacy and security concerns, especially if data is stored by the app provider.
5. Thought-workers typically have diverse workflows and utilize various tools and platforms for their research. Existing note-taking apps may not integrate seamlessly with other tools, causing friction in the workflow and hindering productivity.

### Engineering Challenges

1. Notes are an example of partially structured data with significant requirements on the variety and veracity 'big data' axes:
   - Primarily free-form rich text, but with additional requirements for commenting, tagging, categorising, citing and linking, where such metadata may need to be structured to conform to the thought-worker's or their organisation's controlled vocabularies.
   - Links to other systems such as Data Analytics, Document Management, Project Management; including import and export as well as referencing.
2. Features tend to have been added ad-hoc during the product's lifecycle, resulting in significant technical debt.
3. Specifically, the need for **live collaborative editing** and **system integration APIs** tend to come later in the product's maturity cycle, once customers are already on-board. These are well-known to be hard to retrofit [1].

### Demonstration

The essence of note-taking is sufficiently straightforward that it is frequently used as a demo app for software development tooling. However, such demos are usually oriented to meeting the user's most immediate needs, without addressing requirements that will inevitably become significant later, such as live collaboration and system integration; and particularly data portability, which may even be regarded as a negative feature from the point of view of a software vendor.

## Electronic Lab Notebook

"As a scientist, I want to capture my experimental setup and workflow in a structured and managed but low-overhead way, so that I can easily meet publication and regulatory requirements and have my experiments be reproducible according to the scientific process."

### User Challenges

- Scientific experimental information is inherently variable; novel research requires novel data structures and combinations. Electronic lab notebooks (ELNs) are frequently either too restrictive to cope with novel research (especially if they have been developed for tightly regulated environments), or are so free-form that they amount to little more than a document management system.
- ELN apps store (and may export) data in proprietary formats, making it difficult to migrate or share data across different platforms. Scientists face challenges in accessing and utilizing their data outside of the specific ELN app – this is despite the availability of standards for data export.
- Many scientific projects involve collaboration among multiple researchers and teams. Existing ELN apps may not adequately support collaboration features, making it challenging for scientists to share protocols, notes, and data with collaborators in real-time. (There is potential for huge improvements here, especially when experiments need to change in real-time as they are performed, sometimes very quickly, e.g. if they are automated. [2])
- Scientific research often involves sensitive and proprietary information, including information related to a worker's behaviour. Existing ELNs may have privacy and security concerns, especially if data is stored by the app provider, and especially if that is in an unacceptable legal jurisdiction.
- Scientists use a variety of tools and technologies in their research workflows, including data analysis software, laboratory equipment, inventory systems, and data catalogs. Existing ELN apps may lack seamless integration with these tools, leading to a disjointed workflow and decreased efficiency.

## Engineering Challenges

1. Experimental data are an example of partially structured data with significant requirements on the variety and veracity 'big data' axes:
   - Primarily highly-structured text and numerical data, according to pre-defined standard templates, often modified, and also often not used due to the novelty of the science (but the data is still structured!).
   - Additional requirements for commenting, tagging against controlled vocabularies, and audit logging including electronic signatures.
   - Links to other systems such as Data Analysis, equipment controllers, Lab Informatics including Inventory Systems; including import and export as well as referencing.
2. Features tend to have been added ad-hoc during the product's lifecycle, resulting in significant technical debt.
3. Specifically, the need for **live collaborative editing** and **system integration APIs** tend to come later in the product's maturity cycle, once customers are already on-board. These are well-known to be hard to retrofit [1].

## Demonstration

In this rather more niche use-case, the app engineers are often so subsumed in meeting complex and _inherently_ ever-evolving user requirements, that they are unable to dedicate any time to changing fundamental parts of their system such as architecting to reduce lock-in, provide live collaboration, or switch data formats to more standard ones. In all likelihood, only new projects, e.g. in start-ups, have the opportunity to visit these ideas before it's too late. A valid demonstration including all of these needs at once is difficult, but a senior engineer in this domain could join the dots from a demo of one of the other use-cases.

## bibliography
<br>[1] "Need a lot of endurance" – _How we built Synchrony_, Haymo Meran 2016 https://www.youtube.com/watch?v=2nw2Jx9tSBY
<br>[2] https://m-ld.org/cab