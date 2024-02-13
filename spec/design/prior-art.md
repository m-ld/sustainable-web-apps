# prior art

Existing approaches and trends in computer science and software engineering are addressing subsets of the Sustainable Web Apps principles, and have inspired this project. Here we focus on the contributions of the Semantic Web, Personal Data Stores and Local-First initiatives; and give an indication of what we feel are their individual insights and limitations. Finally we discuss the specific example of the Solid project and community, which aligns closely with our overall intentions.

## semantic web

The "semantic web" refers to an evolved vision of the World Wide Web where information is structured and linked in a way that enables machines to understand and process it more effectively. It aims to make web content more meaningful, interconnected, and accessible to both humans and machines.

![Illustration of the vision for digitization as a meme](https://www.w3.org/comm/assets/stock/shutterstock_728178127-500px.jpg)

Semantic web technologies, such as the Resource Description Framework (RDF) and Web Ontology Language (OWL), facilitate the representation and interlinking of data in a standardised and interoperable manner. This promotes data portability, enabling users to extract and transfer their data between different semantic web applications more easily, thereby reducing lock-in.

The semantic web idea primarily focuses on the organization and structure of information rather than user interfaces or design elements that could lead to attention theft. However the emphasis on open standards and the ability to represent and share data in a decentralised manner aligns with the principle of retaining control over personal data, as users can have greater agency in managing and deciding how their data is stored, accessed, and shared in apps based on the semantic web. We'll discuss this more [below](#solid).

The semantic web encourages the sharing and linking of data across different applications and domains. This suggests the possibility of real-time collaboration. However, it offers no specific approaches, whether abstract or algorithmic, as to how this can be achieved in practice. As a specific example, RDF Collections are not amenable to concurrent editing [[TRUTH-LISTS]], and while this might be reasonably excused as out of scope, the problem needs to be solved somehow.

The core concept of the semantic web revolves around interlinking data across the web. Semantic web technologies provide mechanisms to establish meaningful connections and relationships between disparate datasets and information sources. This aligns with the principle of linking data between applications and facilitates data analysis and knowledge discovery.

Finally, semantic web technologies do not inherently address security concerns. However, by following established best practices and protocols for data management and access control, semantic web applications can incorporate robust security measures.

## personal data stores

With "personal data stores", individuals have control over their own personal data by storing it in a secure and private repository under their ownership and management. This approach aims to address concerns related to data privacy, security, and user control.

Apps based on personal data stores give the user the freedom to switch between different apps or services without losing access to their valuable information. In practice, this also requires that the different apps can read (and maybe write) each other's data; to solve this, personal data stores must be combined with other approaches such as the semantic web.

While the concept of personal data stores primarily focuses on data ownership and control rather than user interfaces, it aligns with the principle of no attention theft indirectly. By enabling individuals to manage their personal data but switch apps at will, personal data stores empower users to have more control over their attention and limit the potential for attention-stealing practices.

Personal data stores, in and of themselves, may not directly facilitate live collaboration, because the data is generally treated as opaque binary objects or files. So again, a combination with other approaches is required to meet this principle.

Personal data stores can support the linking of data between applications by providing standardised APIs or protocols that allow authorised apps or services to access and interact with data stored in the personal data store. However, again, this is inherently at the file level, and this technique must be combined with others to provide more fine-grained links.

Finally, personal data stores specifically prioritise data privacy and security. They can implement robust security measures, such as encryption, access controls, and authentication mechanisms, to safeguard users' data against unauthorised access or breaches. There is another side to this coin, however: large-scale centralised services are known to be targets and therefore dedicate much effort to securing the data under their charge, with associated economies of scale. Whether small providers can do better, even with lower likelihood of attack, is debatable.

## local-first

The concept of "local-first" applications, as canonically described by Ink&Switch [[LOFI]], emphasises prioritizing the user's local device as the primary storage and processing location for data, with synchronization capabilities for collaboration and resilience.

![local-first software empowers collaboration and offline usage by storing data locally and syncing when possible](https://pbs.twimg.com/media/FLVeiv4XEBIH8uK?format=jpg&name=4096x4096)

Local-first applications promote low lock-in by allowing users to store and access their data locally on their devices. Users are not tied to a specific cloud service or platform, enabling them to switch between different applications or services without losing access to their data. Again, switching apps does require that the data is portable; and this is compounded by the promotion of intricate algorithms for device synchronisation like Conflict-free Replicated Data Types (CRDTs).

While local-first applications do not directly address attention theft, they provide a foundation that empowers users to work locally without depending on centralised systems that benefit from their attention.

Local-first applications align strongly with the principle of retaining control over personal data. By storing data locally, users have greater control over their information and can manage its access, sharing, and security.

While the primary focus is on local storage and processing, local-first applications recognise the need for collaboration and provide synchronization mechanisms to enable real-time collaboration among users, as a first-class motivating principle.

The local-first approach does not specifically address linking of data between applications, but research into this is ongoing.

Local-first applications inherently provide a certain level of security by keeping data on the user's local device. Users have direct control over access permissions and can implement security measures tailored to their needs. However, its debatable whether _users_ have the right expertise to manage this. In practice, app developers still have to incorporate best practices for data encryption, secure communication, and user authentication to ensure robust security.

## solid

> Solid is a specification that lets people store their data securely in decentralized data stores called Pods. Pods are like secure personal web servers for data. When data is stored in someone's Pod, they control which people and applications can access it.

Solid specifically combines the concepts of the Semantic Web and Personal Data Stores, thus addressing some of the missing parts of each with respect to sustainability. In particular:

- The semantic web makes no specific commitment to user control over data; in Solid this is provided and guided by the personal data store approach.
- Personal data stores make no specific commitment to data representation (besides at the 'file' level), and so the semantic web approach provides the possibility of both fine-grained linking and improved data portability.

Solid's conceptual model is fundamentally a hub-and-spoke data service architecture, in which data consumers deference URIs to access data in stores ("Pods") on the Web.

![Solid pods](https://solidproject.org/assets/img/solid-pod-tour.svg)

This model de-couples applications from proprietary data storage. Data is instead presented to all apps using a standard interface, and is controlled by the data owner.

The natural (but by no means mandatory) deployment model for this conceptual model is to realise Pods as servers, each with suitable network address translation to allow them to host data addressed with URLs. All Solid implementations are currently server-based.

Therefore, while there is nothing inherently opposed to the local-first approach in Solid, in practice it is not yet addressed. Partly this is because local-first depends strongly on algorithmic techniques for distributed data which allow for the inherent lack of central management.

---

| ≪ prev: [contents](index.md) | next: [hypothesis](hypothesis.md) ≫ |
|------------------------------|-------------------------------------|
