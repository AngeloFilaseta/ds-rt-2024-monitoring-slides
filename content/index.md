
+++

title = "An Architecture and Prototype for Monitoring Distributed Simulations of Distributed Systems"
description = "DS-RT 2024"
outputs = ["Reveal"]
aliases = ["/conference/"]

+++

<!--
    Narrativa:
    INTRO
    - Simulazioni come strumento di testing, perché? 
    - Multiple simulazioni, vengono poi aggregate per ottenre un risultato
    - Problemi, ci vuole un fracco di tempo (mostra processo)
    OBJECTIVEczlc
    - Creare un architettura che permette di visionare aspetti di simulazioni a run time
    HOW TO DO THIS?
    - Monitorare simulazione distributie non è troppo diverso da monitorare un DS classico ma...
    - ci servono le seguenti cose - poter ottenere feedback realtime,
        che i messaggi siano piccoli, poter cambiare cosa osservare velocemente,
        poter monitorare più simulazioni e aggregare i dati
    - filter what to monitor
    ARCHITECTURE
    - Elements
    - How they interact
    - Aggregation... lot of methods in literature
    Prototype
    - Alchemist and GraphQL
    Evalutaion
    - Simulation example
    - DEMO
    - RESULTS
    Future Work

-->

# An Architecture and Prototype for Monitoring Distributed Simulations of Distributed Systems
## DS-RT 2024 - Urbino, Italy, 2024
### Angelo Filaseta - angelo.filaseta@unibo.it
### Danilo Pianini - danilo.pianini@unibo.it
### Angela Cortecchia - angela.cortecchia@unibo.it

---

# Simulations as Testing Tools

- Simulations allow to **validate** theories by comparing outcomes with expected or real-world behavior;
- Usually, **multiple simulations** are run in parallel to obtain a statistically significant result;
  - The results are usually **aggregated** using statistical methods (mean, median, etc.);

{{< figure src="classic.drawio.svg" width="75%">}}

---

# It seems pretty straightforward, but...
###  Ever experienced a little déjà vu in your workflow?
{{% multicol %}}{{% col class="col-8"%}}

- In reality, this process is both **time consuming** and **resource intensive** <small>(and also quite tedious)</small>
- Even under the most favorable conditions,
the time needed to unify the results *is constrained by the slowest execution* among the runs.

- Moreover, we are not accounting for the potential occurrence of **anomalies**, which could not only prolong the time required for the process
  - but may also necessitate starting over entirely! <small>(😭)</small>

{{% /col %}}
{{% col class="col-4"%}}
<img src="wojack.gif" class="wojack"/>
{{% /col %}}
{{% /multicol %}}


---

## Distributed Simulations of Distributed Systems

- An **efficient** and **scalable** method for monitoring *multiple* distributed simulations in real-time is required;

- The objective is to provide **early feedback** about the behavior of the system under development;
  - The system will still be validated using the traditional way, but *a lot of time can be saved* in the long run!<small>(🤩)</small>

{{< figure src="trans.svg" >}}

- Monitoring *distributed simulations* is more complex than monitoring *traditional distributed systems*;
- The parameters to track may not be predefined or known in advance.
  - General-purpose simulators are even more challenging, since parameters may change frequently.

---

# Architecture


{{% multicol %}}
{{% col class="col-7" %}}
### Key Components
-  **Target Distributed System (TDS)**:<br> the simulated element in which properties of interest are located;

-  **Network Node**:<br> the logical device hosting and executing one or more TDSs;

-  **Broker**:<br> provides an interface to interact with a TDS;
    - In this architecture, a broker interacts with only one TDS;
    - three types of interactions: _Query_, _Action_, and _Subscription_;
   
- **Monitor**:<br> interacts with multiple Brokers in order to retrieve data from the TDSs.

{{% /col %}}

{{% col class="mt-5 col-5" %}}
{{< figure src="connections.svg" width="150%">}}
{{% /col %}}
{{% /multicol %}}

---

# A basic implementation

{{% multicol %}}
{{% col class="mt-5 col-7" %}}

- The **Monitor** is a *Web Application*;
  - Thanks to the rich ecosystem of libraries available, data visualization can be effectively implemented;
  
- The **Broker** is a *GraphQL* Server;
  - The GraphQL schema is an ideal choice, providing great flexibility in query construction;
  - It allows to retrieve exactly (and only) the data we need, using the simulator's terminology for intuitive and efficient data access;
  
- Finally, the **TDS** is implemented using *Alchemist*<small>[1]</small>;
  - Alchemist is a general purpose simulator wih a focus on pervasive, aggregate, and nature-inspired computing.

{{% /col %}}
{{% col class="mt-5 col-5" %}}
{{< figure src="architecture.svg" width="150%">}}
{{% /col %}}
{{% /multicol %}}

<small style="text-align: left">
[1] Pianini, D., Montagna, S., & Viroli, M. (2013). Chemical-oriented simulation of computational systems with ALCHEMIST. Journal of Simulation, 7(3), 202–215.</br>
</small>

---

# Evaluation

{{% multicol %}}{{% col class="col-8"%}}
- A morphogenesis process is used as a case study for evaluation.
  - It implements an extended version of the *Vascular Morphogenesis Controller* algorithm in **Aggregate Computing** <small>[2]</small>.
  
- **Nodes** are depicted as circles, while **communication channels** are represented by lines.
- The <strong style=" color:#006666">cyan</strong> and <strong style="color:#6B5000">yellow</strong> areas represent different kinds of resources, attracting the growth of the structure.
- The algorithm begins with a single node and will generate more according to the environmental conditions.
{{% /col %}}
{{% col class="col-4"%}}
{{< figure src="oneroot.gif" width="150%">}}
<div align="center">
Visual evolution of the simulation under observation
</div>
{{% /col %}}
{{% /multicol %}}
<small>
[2] Cortecchia, A., (2024). Multiplatform Self-Organizing Systems Through a Kotlin-MP Implementation of Aggregate Computing. International Conference on ACSOS 2024.</br>
</small>

<small>Our reproducibility suite is available on GitHub :  [AngeloFilaseta/dsrt-2024-distributed-monitoring](https://github.com/AngeloFilaseta/dsrt-2024-distributed-monitoring) </small>

----

# Evaluation - Scalability

- A **Baseline Query** retrieve the whole state of the simulation , while the **Specific Query** retrieves a subset of data.

- The size of the response is checked for both queries as the number of nodes increase in the simulation.
    - In both approaches, the response size scale proportionally with the node count;
    - However, the **Specific Query** is more efficient, as it retrieves only the required data in *real-time*.

{{< figure src="scalability.svg" height="20">}}
<small>Results reflect the aggregation of *10* simulations, each with a different seed.</small>

---

# Evaluation - Efficiency

- **Lost updates**:  a missed event that occurred between two consecutive query calls, in which the target measure changed one or more times, but it was not captured
- **Useless polling**: a query is executed without capturing new events since the last executed query.

{{< figure src="efficiency.svg" height="20">}}

<small>Results reflect the aggregation of *100* simulations, each with a different seed.</small>

---

# Evaluation - Viability

- It is possible to monitor the evolution of data of multiple distributed simulation in real-time.
- A very basic aggregation algorithm has also been implemented.
    - It helps in identifying the overall trends of all the simulations.

---

# Future Work

---

# Thank you for your attention!

{{< figure src="questions.webp" width="15%">}}

#### Some Media Attributions are required

- [Flaticon](https://www.flaticon.com/) has been used to retrieve some icons: credits to *Freepik*;
- [Giphy](https://giphy.com/) has been used to retrieve some gifs and stickers, credits to *WimpyKid*.

---
