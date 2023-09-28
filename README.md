This is a working example of a backend that can allow [GGRN](http://github.com/ekernf01/GGRN) to interface with any GRN software that can run in a Docker container. 

### Explanation

- **Basics:** Create a Docker image ([example Dockerfile](https://github.com/ekernf01/ggrn_docker_backend/blob/master/Dockerfile)). Upon running, software in the container should do the following. (You can reuse the boilerplate from our example in [train.py](https://github.com/ekernf01/ggrn_docker_backend/blob/master/train.py).
    - read training data from `to_from_docker/train.h5ad`. You can expect this training data to pass the checks in `ggrn.validate_training_data(train)`.
    - read perturbations to predict in `to_from_docker/perturbations.json`. You can expect a list of lists like `[["NANOG", 5.43], ["KLF4", 6.78]]`, meaning you should predict one observation where NANOG expression is set to 5.34 and another where KLF4 expression is set to 6.78. For multi-gene perturbations, you'll find comma-separated lists as strings, and yes, I'm very sorry about this. An example: `[["NANOG,POU5F1", "5.43,0.0"], ["KLF4,SOX2", "6.78,9.12"]]`. 
    - train your method and make those predictions, preserving the order.
    - save the predictions in `h5ad` format as `to_from_docker/predictions.h5ad`. To be ultra-safe about not scrambling them, the order is expected to match what you find in `perturbations.json`, and the `.obs` is expected to have a column `perturbation` and a column `expression_level_after_perturbation`.
- **Passing GGRN args to your method**: When using GGRN with a Docker backend, you can still pass all [the usual GGRN args](https://github.com/ekernf01/ggrn/blob/main/GGRN.md) to `GRN.fit()`. These will be saved to `to_from_docker/ggrn_args.json` and mounted to the container, so your code can look for keyword args in `to_from_docker/ggrn_args.json`. 
- **Passing custom args to your method**: When using GGRN, you can pass a dict `kwargs` to `GRN.fit`. This will be saved to `to_from_docker/kwargs.json` and mounted to the container, so your code can look for keyword args in `to_from_docker/kwargs.json`. Be aware that Python may not translate perfectly to json; for instance, json lacks Python's `None` value.
- **Passing args to docker**: When you call GGRN, you will specify `method="docker__myargs__myimage"` when you call `GRN.fit`. The second element of this double-underscore-separated list is provided directly to docker, as in `docker run myargs <other relevant stuff> myimage`. For example, to limit the cpu usage, you can use `method="docker__--cpus='.5'__myimage"`, or to use a gpu, you can use `method="docker__--gpus all ubuntu nvidia-smi__myimage"`. Don't use `--rm` or `--mount`; we already use those args. We have not tested whether there are other things you could provide that would interfere with how we call Docker.

### Example

To try it out, install [GGRN](https://github.com/ekernf01/ggrn) and obtain some [test data](https://github.com/ekernf01/perturbation_data), then run the following. We recommend setting up your environment as documented in our [benchmarking project](http://github.com/ekernf01/perturbation_benchmarking).

```python
import ggrn.api as ggrn
import load_perturbations
# Obtain example data
load_perturbations.set_data_path(
    '../perturbation_data/perturbations' # Change this to where you put the perturbation data collection.
)
train = load_perturbations.load_perturbation("nakatake")
grn = ggrn.GRN(train) 
# This line saves some inputs to `to_from_docker`, but doesn't actually run the container.
grn.fit(
    method ="docker__--cpus='.5'__ekernf01/ggrn_docker_backend", 
    kwargs = {"example_kwarg":"george"}                    
)
# This line runs the container, doing both training and prediction, then removes the container.
predictions = grn.predict([("POU5F1", 0), ("NANOG", 0)])
# You should be left with an AnnData. 
predictions
```



