# Individual Assignment 5: Explainability

(17-445/17-645 Machine Learning in Production)

## Overview

In this assignment you will consider how to explain a model and its decisions to an internal reviewer and to end users. You will prototype global explanations and individual explanations. 

Learning goals:

* Understand and distinguish the different notions of transparency, interpretability, and explainability
* Explore explainability needs from the perspective of the end user
* Use state-of-the-art tools for model explainability to generate model and individual explanations
* Consider the need of end users, not just developers
* Interpret regulation and provide evidence to demonstrate compliance. Reflect on the difficulty of working with provided policies.



## The Use Case: Diabetic Retinopathy Screening

The use case is to detect [diabetic retinopathy](https://www.nei.nih.gov/learn-about-eye-health/eye-conditions-and-diseases/diabetic-retinopathy) early with a special purpose ML-powered medical device. Diabetic retinopathy is a condition that can result in vision loss if left untreated. [AEYE Health](https://www.aeyehealth.com) is a comparable commercial offering.

Consider you are building a smartphone app, replacing current large commercial devices. The app will be used by trained users (nurses, volunteers) to perform screenings to detect potential problems simply by having patients look into the smartphone's camera through a specialized lens attachment (a small 3-d printed holder for a lens).

Deploying such medical diagnostics as a smartphone app and low-cost hardware extension has the potential to drastically reduce screening costs and make screenings much more available, especially in underresourced regions of the world. Instead of having to travel to a clinic with specialized equipment, trained users could perform screenings at mobile clinics or in patients' homes. The app would provide information about a potential risk and encourage the patients to get in contact with medical professionals for more accurate testing and potential treatment.

In this assignment, you will focus not only on the model, but also consider its integration into a smartphone app and its use for screening by trained users in underresourced regions (e.g., remote areas, high-poverty areas).

## The Policy

While government regulation is still evolving, the parent company of the organization that creates the smartphone app has set an internal responsible AI policy to hold itself to high standards of responsible engineering. The policy is mandatory for all projects using machine learning (similar to what many other big companies in the tech and medical space have done). The following 8 points are the relevant parts of the policy:

**INTENDED USE** 

1. Describe the automated system’s intended use and the role of the automation (model). [*Guidance: Include its purpose, setting of use, and intended user(s). Include information about how the automated system may be used in combination with other information.*] 
2. Provide evidence that the automation (model) functions accurately, consistently, and effectively in the intended use case. [*Guidance: Evidence could take the form of accuracy testing disclosures, data from real-life tests of the tool, or independent review.*]

**HOW IT WORKS** 

3. Describe how the automation (model) works generally. Provide evidence that the documentation is effective for the policy purpose. [*Guidance: Where possible identify general mechanisms or factors that most strongly influence the automation.*]

4. Provide a mechanism to describe how the automation (model) worked with regard to an instance of use to all intended users and subjects affected by the automated system in a form that is accessible to them. Descriptions must include (1) that automation was used, (2) what significant personal data was used for the decision, and (3) what decisions were reached in a specific case. [*Guidance: Provide individual explanations using appropriate mechanisms, such as graphs, text-based explanations, counterfactuals, or SHAP plots.*]

**CONCERNS**

5. Describe limitations and misuse potential of the automated system beyond its intended purpose and any provided mitigations. [*Guidance: Identify concrete problems and assess their risks.*]

6. Describe the data used by the automated system. Justify the use of personal identifiable information.

7. Describe how to report misuse or harm from the automated system.

**LANGUAGE REQUIREMENTS**

8. Provide all documentation in language appropriate for the intended audience. All documentation for untrained users must use nontechnical language at an *eighth grade reading level*.

## Stakeholders

You will provide explanations for four stakeholders: 

* Global explanations for an ML engineer on our team who reviews the model before release: does it rely on medically plausible factors, does the patient data help, and does it behave reasonably across patient groups?
* Global explanations for procurement officers in non-profit organizations who make decisions about buying the product
* Individual explanations for nurses or volunteers who perform the screening with patients
* Individual explanations for patients who are receiving the screening

## Tasks

In this assignment, you will work with a provided model for a potential medical application ([diabetic retinopathy detection with a smartphone app](https://github.com/cmu-seai/diabetic-retinopathy)), understand explanation needs, create explanations for four different stakeholders, and demonstrate compliance with policy requirements.

While we only provide the model and training/test data, assume that the model is embedded in a real-world software product as described above. 

**Understanding explainability needs.** Consider what the four stakeholders might want to know about the product, the model, or the data (transparency). Consider their goals, such as understanding the system (as needed), using it responsibly, answering patient questions, or providing effective oversight ("human in the loop"). 

We expect that you know what the ML engineer might want but for the others, consider using personas, chatbots with personas, hazard analysis, or interacting with actual people with experience in this role to understand their needs. Consider the model in the context of the software system and consider how that software system is embedded in the larger medical context. You likely will want to better understand the disease and the medical process for screening and treatment first, as well as challenges with deploying this in underresourced and remote areas.

**Global explanations for the ML engineer:** To meet the previously identified explanation needs of the first stakeholder, create a file `explanation_developer.md` or `explanation_developer.pdf` with an internal review of the model before release. This document is for internal use only, to be read by an ML engineer who was not the original model developer. You may inspect individual examples to support a finding, but each finding must be about the model's overall behavior. You will write code to produce the analyses for this document. In `explanations.md`, argue why your solution is suitable for this stakeholder and their needs.

**Global explanations for procurement officers:** To meet the previously identified explanation needs of the second stakeholder, create a file `explanation_procurement.md` or `explanation_procurement.pdf` that includes information in a form that the company producing the product *may disclose in its handbook or website or advertising materials*. This might include information about the data, about accuracy, about important features, or about fairness -- you will likely write some code to produce relevant information for this document. You do not need to make up unrelated parts of the handbook/website/leaflet that have nothing to do with the model, such as how to turn on the device. In `explanations.md`, argue why your solution is suitable for this stakeholder and their needs.

**Individual explanations for nurses and patients:** Create explanations that you could give to the third and fourth stakeholder *on a display or as a printed handout after the screening*. In `explanations.md`, argue why your solution is suitable for the intended stakeholders and their needs. Write code that produces explanations *for a specific input* for a patient, for example by generating HTML or PDF pages. Create the files `explanation_nurse.md` or `explanation_nurse.pdf` and `explanation_patient.md` or `explanation_patient.pdf` (generated with code) that each show the explanations for the same *two* patients (example inputs) -- that is, concatenate the explanations for two selected patients in the same file. You can use the same explanations for both stakeholders if you think that is appropriate. Provide brief instructions on how to run the code to recreate the submitted explanations and to create explanations for other inputs.

**Policy compliance:** In a file `compliance.md`, discuss whether your product, with the explanations you created for stakeholders 2 to 4 taken together, complies with the provided policy. The internal developer review is not part of this compliance discussion. Write a single discussion that covers all three stakeholders together, but discuss each of the eight policy requirements separately. Explain whether your solution complies, and where possible provide evidence for compliance -- what you consider as evidence is for you to decide. Note that compliance is not necessary to receive full credit for this assignment. If you think that your solution does not comply or a policy requirement is not applicable, explain why and either (a) outline what additional work you could do to comply, (b) explain why compliance is not possible, or (c) why the policy requirement is not applicable. 

**Reflection:** Reflect on your experiences in this assignment, what you considered challenging, and what additional steps you might take if you had to demonstrate policy compliance to a government agency (e.g. FDA). 



### Hints and guidance

A key goal is to provide explanations for the specified target audience and their needs when this system is deployed. Consider what these people might want to know and what they are likely to understand. Actively try to understand their perspective and how the model fits into the software and how that software fits into the medical system more broadly. You may want to conduct some internet research, talk to relevant people, or brainstorm with a chatbot. We strongly recommend trying to talk to potential patients without AI expertise. The goal is *not* to merely provide explanations that you wish *you* would receive or explanations for developers to debug the model (except for the first stakeholder), but explanations that are useful for each listed stakeholder. Less may be more.

This assignment is intentionally open ended. We interpret explainability broadly, including technical post-hoc explanations like SHAP, local and global explanations, nontechnical textual descriptions and traditional transparency mechanisms, such as audits and model and data disclosures. We have no specific requirements about approaches or techniques to include; make a judgment about what you think is needed and understandable to the intended target audience.

You can explore many different explainability and transparency techniques, including the ones discussed in class and in the lab and ones you find on your own. We recommend to rely on existing explainability tooling for nontrivial analyses rather than to develop your own. Many existing tools can generate plots that you can integrate into the reports you generate, but textual explanations are also perfectly acceptable. Think about what the target audience needs. We do not care about the visual quality of the generated files.

There is no single right explanation and we expect to see a very wide range of very different solutions. We are looking for a plausible solution where you argue why this solution is suitable and compliant, not any specific tool or design. 

## Deliverable

See Canvas for instructions of how to create a private repository with GitHub classroom that contains the starter code. Submit the link *to your last commit* on GitHub to Canvas. The URL must be in the format `https://github.com/cmu-seai/[repo]/commit/[commitid]`. Make sure that the link includes the long ID of the last commit. The code and various markdown files at this commit is what we will grade.

**Code:** Commit all relevant code to your GitHub repository, but do not commit private credentials. 

**Explanation needs (3 pages/1500 words max):** In a file `explanation_needs.md` in the root directory of the repository, describe the steps you took to identify explanation needs and provide some corresponding evidence (e.g., links to resources considered, links to chatbot transcripts, hazard analysis steps, personas). Separately describe the explanation needs you identified for each of the four stakeholders.

**Explanations:** Include the four files `explanation_developer.md/pdf`, `explanation_procurement.md/pdf`, `explanation_nurse.md/pdf`, and `explanation_patient.md/pdf` in the root directory of the repository; these files have no page limit. Additionally, in a file `explanations.md` (3 pages/1500 words max) in the root directory of the repository, briefly describe for each stakeholder what kind of explanations you created and link them to the previously identified explanation needs. The explanations for different stakeholders may share content where appropriate. Point out where we can find each explanation in the corresponding file if it is not obvious. Provide links to code for creating the data or visualization used and point out specific techniques or tools, if you used any. Explain how we can recreate the submitted individual explanations and produce individual explanations for other patients with your code.

**Compliance (4 pages/2000 words max):** In a file `compliance.md` in the root directory of the repository, discuss whether your product, with the explanations for stakeholders 2 to 4 taken together, complies with the policy requirements. If possible, provide evidence to demonstrate compliance. Write this as if you were submitting a short report to a compliance/legal team in your company to convince them that your product complies and can be released. If you cannot comply, think a policy rule is not relevant, or are unsure, discuss additional steps needed for compliance or explain why compliance may not be possible/relevant.

**Explanation/reflection:** Within 2 weeks of submitting the assignment meet with a member of the course staff during office hours to explain your solution. We may ask questions about your approach to identify information needs, your explanations, your implementation, and your policy compliance. In addition, we may engage you on the following questions: What parts of this assignment and the policy were challenging? How would you approach policy compliance when it came to a government policy (e.g. FDA)?


Page limits are recommendations and not strictly enforced. You can exceed the page limit if there is a good reason. We prefer precise and concise answers over long and rambling ones.

## Grading

The assignment is worth 100 points. For full credit, we expect:

* Assignments with an invalid link (e.g., without a commit ID) will receive 0 points.
* [ ] 10 points: A file `explanation_needs.md` in the root directory of the linked repository describes the identified explanation needs separately for each of the four stakeholders. The process for identifying the information needs is described and some evidence is provided that the process was followed.
* [ ] 20 points: There are explanations in the files `explanation_developer`, `explanation_procurement`, `explanation_nurse`, and `explanation_patient` (each `.md` or `.pdf`) in the root directory of the linked repository. Each explanation is clearly designed for the stakeholder, addressing the needs of the stakeholder, using appropriate language. A file `explanations.md` in the root directory of the linked repository explains for each stakeholder how the explanations meet the previously identified explanation needs.
* [ ] 20 points: The explanations use appropriate explainability approaches, with a clear distinction between global explanations for stakeholders 1 and 2 and individual explanations (for two patients) for stakeholders 3 and 4. The file `explanations.md` names the techniques used.
* [ ] 10 points: Code is provided and linked from `explanations.md`. The code produces the submitted explanations and can produce the individual explanations for other patients. The file `explanations.md` contains working instructions on how to recreate your individual explanations and to create individual explanations for other patients with the implementation.
* [ ] 20 points: A file `compliance.md` in the root directory of the linked repository discusses each of the eight policy requirements separately, once for the explanations provided to stakeholders 2 to 4 taken together. A requirement counts as addressed if (a) the answer describes how the requirement was addressed technically, (b) the provided explanation files illustrate the solution, and (c) the answer makes a good faith attempt at providing evidence to demonstrate compliance, or discusses additional steps needed for compliance, or explains why compliance may not be possible or applicable.
  * 10p: at least 6 of the 8 requirements are addressed
  * 20p: all 8 requirements are addressed
* [ ] 20 points: You can convince the course staff during office hours within 2 weeks of submitting your solution that you understand your solution and that you have engaged with the reflection questions.

