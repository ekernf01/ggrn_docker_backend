This is a working example of a backend that can allow [GGRN](github.com/ekernf01/GGRN) to interface with any GRN software that can run in a Docker container. To try it out, install GGRN and obtain some test data, then run the following. One way to set up your environment is documented in our [benchmarking project](github.com/ekernf01/perturbation_benchmarking).

```python
import ggrn.api as ggrn
import load_perturbations
# Obtain example data
load_perturbations.set_data_path(
    '../perturbation_data/perturbations' # Change this to where you put the perturbation data collection.
)
train = load_perturbations.load_perturbation("nakatake")
report = ggrn.validate_training_data(train) 
grn = ggrn.GRN(train) 
# This line saves some inputs to `to_from_docker`, but doesn't actually run the container.
grn.fit(
    method = method="docker|--cpus='.5'|ekernf01/ggrn_docker_demo", 
    kwargs = {"example_kwarg":"george"},                      
)
# This line runs the container, doing both training and prediction, then removes the container.
predictions = grn.predict({("POU5F1", 0), ("NANOG", 0)})
```



