
# Kosli Core Concepts

## What are Kosli Flows and Trails?

- A Kosli Flow represents a business or software process for which you want to track changes and monitor compliance.
- A Kosli Trail represents a single execution of a process represented by a Kosli Flow. 
- Each trail must have a unique identifier of your choice, based on your process and domain. 
- Example identifiers (for software processes) include git commits or pull request numbers.

## What are Kosli Environments and Snapshots?

- Environments in Kosli monitor changes in your software runtime systems.
- You create a Kosli Environment for each physical or virtual runtime environment you want to track.
- An environment snapshot represents the reported status (running artifacts) of your runtime environment at a specific point in time. 
- Snapshots are immutable, append-only objects. Once a snapshot is created, it cannot be modified.
- In each snapshot, Kosli links the running artifacts to the Flows that produced them. 
- Snapshot compliance relies on the compliance status of each running artifact, while environment compliance depends on its latest snapshot compliance.

# cyber-dojo

There is a public [Kosli Organization called cyber-dojo](https://app.kosli.com/cyber-dojo/dashboard/) which you can explore
without having to log into Kosli. It is the Kosli Organization for [cyber-dojo](https://cyber-dojo.org), an open-source
application for practicing TDD from your browser. 

## Explore Kosli Flows and Trails for cyber-dojo

- Cyber-dojo has 10 microservices, each with their own repository.
- There is a Kosli Flow for each repository's CI pipeline.
- Each Kosli Flow contains a Kosli Trail for each commit to its corresponding repository. For example:
  - [runner-ci](https://app.kosli.com/cyber-dojo/flows/runner-ci/trails/) is the Kosli Flow for the
    [runner](https://github.com/cyber-dojo/runner) repo's CI pipeline (on GitHub). 
    - [1394fe76d45aaf40bf19817e0d8110b570848c9f](https://app.kosli.com/cyber-dojo/flows/runner-ci/trails/1394fe76d45aaf40bf19817e0d8110b570848c9f)
    is the Kosli Trail for the *runner* Artifact built from commit [1394fe](https://github.com/cyber-dojo/runner/commit/1394fe76d45aaf40bf19817e0d8110b570848c9f).
    - This Trail has numerous pieces of evidence (attested from its CI pipeline), including 
    a [snyk-code-scan](https://app.kosli.com/cyber-dojo/flows/runner-ci/trails/1394fe76d45aaf40bf19817e0d8110b570848c9f?attestation_id=07046aeb-9e1f-43cd-b68b-d8a7f0ae).
  - [creator-ci](https://app.kosli.com/cyber-dojo/flows/creator-ci/trails/) is the Kosli Flow for the
    [creator](https://gitlab.com/cyber-dojo/creator/) repo's CI pipeline (on Gitlab). 
    - [2252c4c22d325c5da618f90744625e540fc7cfae](https://app.kosli.com/cyber-dojo/flows/creator-ci/trails/2252c4c22d325c5da618f90744625e540fc7cfae)
    is the Kosli Trail for the *creator* Artifact built from commit [2252c4c](https://gitlab.com/cyber-dojo/creator/-/commit/2252c4c22d325c5da618f90744625e540fc7cfae). 
    - This Trail also has numerous pieces of evidence (attested from it CI pipeline), including 
    a [pull-request](https://app.kosli.com/cyber-dojo/flows/creator-ci/trails/2252c4c22d325c5da618f90744625e540fc7cfae?attestation_id=9aac53fa-58ac-46d6-b20a-ee2dc4c7).

## Explore Kosli Environments and Snapshots for cyber-dojo

- Each cyber-dojo repo CI pipeline deploys to two AWS ECS clusters:
  - https://beta.cyber-dojo.org runs on its staging cluster. 
    - The Kosli Environment for this cluster is [aws-beta](https://app.kosli.com/cyber-dojo/environments/aws-beta/events/)
  - https://cyber-dojo.org runs on its production cluster.
    - The Kosli Environment for this cluster is [aws-prod](https://app.kosli.com/cyber-dojo/environments/aws-prod/events/)
- Each Kosli Environment page has two main tabs:
  - [Snapshots](https://app.kosli.com/cyber-dojo/environments/aws-prod/snapshots/)  
    Each snapshot is numbered (from 1) and shows all the Artifacts running at a given moment in time and their compliance status.
    At the time of writing, there are 2793 snapshots for `aws-prod`, covering several years. 
  - [Log](https://app.kosli.com/cyber-dojo/environments/aws-prod/events/)  
    The log shows all the changes to individual Artifacts (and their compliance status) in the given Environment. 
    The log is paginated, and at the time of writing there are 131 pages for `aws-prod`.
