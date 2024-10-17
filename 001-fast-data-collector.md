# Fast Data Collector

Recently I developed an experimentation catalog. There are 2 main use-cases: 

- WRITE: An evaluation service will be writing results to the catalog with potentially high concurrency. When an experiment is run, many writes will come in over a short period of time.

    An example record might look like this: 

    ```json
    {
        "projectId": "project-1",
        "experimentId": "experiment-1",
        "runId": "run-1",
        "metrics": {
            "accuracy": 0.95,
            "precision": 0.85,
            "recall": 0.75
        }
    }
    ```

- READ: Users will interact with the catalog to view the results of the evaluations. You can assume reads will get all records for a given run.

One of the off-the-shelf solutions we tested had difficulty scaling to the write load when many replicas were sending data, so I decided to call this challenge "Fast Data Collector" though of course, we also need to read the data at a reasonable speed.

## Challenge 1

Design the storage solution for the catalog adhering to the following requirements:

- Organization: The catalog will contain a list of projects. Each project will contain a list of experiments. Each experiment will contain a list of runs. Each run will contain a list of results.

- Technology: Use Azure Storage to reduce cost and complexity.

- Scalability: thousands of results per run, dozens of runs per experiment, dozens of experiments per project.

- Performance: Be able to write a 200 results per second. Be able to read all results in an experiment in 2 seconds or less.

The design should address at least the following:

- How is data partitioned? Why?

- How are the read and write performance requirements met?

- Are there any consistency concerns?

- What are the limitations and challenges of the design?

- What is the complexity of implementation and maintenance?

## Challenge 2

Design the storage solution for the catalog ignoring the Technology constraint and make a case for using a different technology.