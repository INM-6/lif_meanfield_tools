**The development of the toolbox continues in a new repository: [NNMT](https://github.com/INM-6/nnmt)**
----------------------------------------------------------------------------------------------------------------------

LIF Meanfield Tools
====================

This Python package provides tools for analyzing neuronal networks consisting of leaky integrate-and-fire (LIF) neurons. These tools are based on
mean-field theory of neuronal networks. That is why this package is called
__lif_meanfield_tools (LMT)__.

The package provides implementations used in the same or a similar version in
the following scientific publications: [Fourcaud & Brunel (2002)](https://doi.org/10.1162/089976602320264015),
[Schuecker et al. (2014)](https://arxiv.org/abs/1410.8799),
[Schuecker et al. (2015)](https://doi.org/10.1103/PhysRevE.92.052119),
[Schuecker et al. (2017)]( https://doi.org/10.1371/journal.pcbi.1005179),
[Bos et al. (2016)](https://dx.doi.org/10.1371%2Fjournal.pcbi.1005132), and
[Senk et al. (2020)](https://journals.aps.org/prresearch/abstract/10.1103/PhysRevResearch.2.023174).

Using this package, you can calculate quantities like firing rates, power
spectra, and many more, which give you a deeper and more intuitive
understanding of what your network does. If your network is not behaving the
way you want it to, these tools might help you to figure out, or even tell you,
what you need to change in order to achieve the desired behaviour. The toolbox
allows storing results and reuse them for further analyses.

If you are using this toolbox, please cite us: For a specific release, we recommend to use the reference from [Zenodo](https://zenodo.org/). Otherwise, you can also provide a link to this repository with the hash of the respective commit.
In addition, please also cite the publications that used the methods implemented here first. In [How to Use This Package](#how-to-use-this-package) you can find details on which function of this package refers to which publication.

<img src="https://github.com/INM-6/lif_meanfield_tools/blob/master/readme_figures/power_spectra.png" width="400">

The figure shows power spectra calculated with this toolbox using the minimal
example script `examples/power_spectra.py`.


# Structure

<img src="https://github.com/INM-6/lif_meanfield_tools/blob/master/readme_figures/structure_new.png" width="400">

lif_meanfield_tools consists of four modules:

- The central module is __network.py__. It defines a class `Network` which is a
  container for network parameters, analysis parameters and calculated results.
  `Network` comes with all the methods that can be used to calculate network
  properties, like firing rates or power spectra. Additionally, there are some
  'administrative' methods for changing parameters or saving.

- __input_output.py__ is called by `network.py` for everything that is related
  to input or output. Here we defined saving and loading routines, quantity
  format conversions and hash creation.

- __meanfield_calcs.py__ is the module which is called every time a mean-field
  related method of `Network` is called. Here we put all the mathematical
  details of the mean-field theory.

- __aux_calcs.py__ is a module where auxiliary calculations that are needed in
  `meanfield_calcs.py` are defined. These functions are supposed to be generic,
  non-specific building blocks. However, it is difficult to draw a line between
  the calculations that belong to `meanfield_calcs.py` and the ones that belong
  to `aux_calcs.py`.

- __utils.py__ is a module which contains utility functions. It contains
  functions that simplify working with pint quantities and decorators for
  checking that passed parameters are in valid regimes.

# How to Get Started / Installation

If you have a local copy of this repository, you can install LMT by running:
```
pip install .
```

An alternative is to install directly from GitHub:
```
pip install git+https://github.com/INM-6/lif_meanfield_tools.git
```
If you are using [conda](https://conda.io/), you can create an environment including LMT:
```
conda env create -f environment.yaml
```

# Current Issues

As the package is still maturing, we currently have some issues that you should
be aware of:

- [__Network model used__](https://github.com/INM-6/lif_meanfield_tools/issues/36):
  Currently, the toolbox is specialized on the microcircuit model (first
  published by [Potjans and Diesmann (2014)](https://doi.org/10.1093/cercor/bhs358))
  which is why the network parameter `label` should only be set to
  `microcircuit` at the moment. See the function
  `_calculate_dependent_network_parameters()` in `network.py`.

- [__Accuracy of transfer function at high frequencies__](https://github.com/INM-6/lif_meanfield_tools/issues/37):
  This actually is not a real issue, but you should be aware that the current
  implementation is only accurate for moderate frequencies. This is expected
  from the theory implemented. In the future we might add a support for high
  frequencies (see [Schuecker et al. (2015)](https://doi.org/10.1103/PhysRevE.92.052119)
  for further discussion).

- [__Accuracy of transfer function depends on ratio of synaptic and membrane time constant__](https://github.com/INM-6/lif_meanfield_tools/issues/37): This
  is a part of the theory as well. It is only accurate for small values of
  <img src="https://render.githubusercontent.com/render/math?math=\sqrt{\tau_s/\tau_m}">
  which is used as a perturbation parameter in the analysis (see
  [Schuecker et al. (2015)](https://doi.org/10.1103/PhysRevE.92.052119) for
  further discussion).

# How to Use This Package

In order to give you a quick and simple start, we wrote some example scripts in
the folder `examples`. You can start with `minimal_usage_example.py`. First of
all, you should have a look at this file. Actually, we hope that the usage
might be self-explanatory, once you have seen an example. But, if you need a
little more hints, just continue reading.

For using LMT, you need to store all your network parameters and your analysis
parameters in .yaml files, as we have done it for the example script. If you
don't know how the .yaml file format works, you could either first read
something about it, or use our example .yaml files as templates.

So, let us start coding. First of all you need to import the package itself.
Additionally, you might want to define a variable to store the `pint` unit
registry (ureg). This is needed for dealing with units and some of the
functionality implemented needs the usage of pint units.

Now, you can instantiate a network by calling the central LMT class `Network`
and passing the .yaml file names. A `Network` object represents your network.
When it is instantiated, it first calculates all the parameters that are
derived from the passed parameters. Then, it stores all the parameters
associated with the network under consideration. Additionally, it checks
whether these parameters have been used for an analysis before, and if so loads
the corresponding results. Newly calculated results are stored withing the
`Network` object as well.

A `Network` object has the ability to tell you about it's properties by calling
the corresponding method as
```
	network.<property>()
```
Here, `<property>` can be replaced by lots of stuff, like for example
`firing_rates`, `transfer_function`, or `power_spectra`. You can find the
complete list of `Network` methods at the end of this section. When such a
method is called, the network first checks whether this quantity has been
calculated before. If so, it returns the stored value. If not, it does the
calculations, stores the results, and returns them. If you want to calculate
properties that need another quantity to be calculated first, you will receive
an error message telling you which quantities to calculate first. For example
the `power_spectra` need the `delay_dist_matrix` and the `transfer_function`,
which in turn needs the `working_point` So a correct call would look like
```
  network.working_point()
  network.transfer_function()
  network.delay_dist_matrix()
  network.power_spectra()
```
Functions like `transfer_function` will use the previously calculated results.
This allows you to use different methods for calculating the same quantities
(like `taylor` and `shift` for the firing rates and the tranfer function) in a
building block like fashion.

Sometimes, you might want to know a property for some specific parameter, like
for example the `power_spectra` at a certain frequency. Then, you need to pass
the parameter including its unit to the method, e.g.,
```
	network.power_spectra(10 * ureg.Hz)
```
If you want to save your results, you can simply call
```
	network.save()
```
and the calculated results, together with the corresponding parameters, will be
stored inside a .h5 file, whose name contains a hash, which reflects the used
network parameters.

Network methods:
- __save__: Save all calculated results together with network and analysis
  parameters into an .h5 file.
- __show__: Return a list of quantities that have already been calculated.
- __change_parameters__: Create a new instance of Network class with adjusted
  specified parameters.
- __firing_rates__: Calculate the firing rates in a self-consistent mean-field
  manner. The algorithm starts with firing rate zero for all populations, then
  calculates the resulting mean and variance of the input to a neuron, and uses
  the results and Eq. (4.33) in
  [Fourcaud & Brunel (2002)](https://doi.org/10.1162/089976602320264015)
  to calculate the resulting firing rate again. This procedure is continued
  until the rates converge.
- __mean_input__: Calculate mean input to a neuron, given the population firing
  rates and external inputs.
- __std_input__: Calculate the standard deviation of the input to a neuron,
  given the population firing rates and external inputs.
- __working_point__: Return firing rate, mean and standard deviation of input.
- __delay_dist_matrix__: Compute a matrix of prefactors in frequency domain
  dependent on a given delay distribution.
- __transfer_function__: Calculate the transfer function following Eq. (93)
  in
  [Schuecker et al. (2014)](https://arxiv.org/abs/1410.8799)
  in first order perturbation theory in
   <img src="https://render.githubusercontent.com/render/math?math=\sqrt{\tau_s/\tau_m}">,
  the square root of the synaptic time constant divided by the membrane time
  constant. You can choose between two implementations:
  `taylor` and `shift`. The difference is the way the colored noise is treated
  mathematically, which leads to two slightly different  approximations, which
  are however equivalent up to first order (see
  [Schuecker et al. (2015)](https://doi.org/10.1103/PhysRevE.92.052119)
  for further discussion).
- __sensitivity_measure__: Calculate the sensitivity measure, introduced in
  Eq. (7) of
  [Bos et al. (2016)](https://dx.doi.org/10.1371%2Fjournal.pcbi.1005132),
  which can be used to identify the connections
  crucial for the peak amplitude and frequency of network oscillations, visible
  in the power spectrum.
- __power_spectra__: Calculate the power spectra of all populations following
  Eq. (18) in [Bos et al. (2016)](https://dx.doi.org/10.1371%2Fjournal.pcbi.1005132).
- __eigen_spectra__: Calculate the eigenvalue spectrum, or left of right
  eigenvectors of the effective connectivity matrix (Eq. 4), the propagator
  Eq. (16) or the inverse propagator in the frequency domain as defined in
  [Bos et al. (2016)](https://dx.doi.org/10.1371%2Fjournal.pcbi.1005132).

The following additional Network methods have been used in [Senk et al. (2020)](https://journals.aps.org/prresearch/abstract/10.1103/PhysRevResearch.2.023174):
- __additional_rates_for_fixed_input__: Compute external excitatory and
  inhibitory rates to obtain a fixed working point (see Appendix F).
- __fit_transfer_function__: Fit the transfer function with a low-pass filter
  (see Fig. 5(b) and (c)).
- __scan_fit_transfer_function_mean_std_input__: Iterate different combinations
  of mean and standard deviation of input using `fit_transfer_function()`
  (see Fig. 5).
- __effective_coupling_strength__: Compute the effective coupling strength
  according to Eq. (E1).
- __linear_interpolation_alpha__: Linear interpolation between LIF transfer
  function and low-pass filter (see Fig. 6),
- __eigenvals_branches_rate__: Compute eigenvalues for branches of the
  Lambert W function corresponding to the analytically exact solution of the
  neural-field model (see Fig. 6 for alpha=0).
- __xi_of_k__: Effective spatial profile (see Fig. 3(b) and (d)).
- __solve_chareq_rate_boxcar__: Analytical solution of the characteristic
  equation for a neural-field model with boxcar-shaped connectivity kernels.

# Testing

We have an extensive test suite using the `pytest` framework. If you want to
run all the tests, you can do so by installing and activating the conda
environment specified in the provided `environment.yaml` file, and running
```
pytest
```
from the root directory (the one containing `tests` and `lif_meanfield_tools`).
If you want to be more specific, you can run single tests as well
```
pytest tests/unit/test_meanfield_calcs.py::Test_firing_rates::test_correct_output
```
See the `pytest` documentation for all available options.

Note that some tests currently fail. This points towards pieces of code that
still need to be improved (see current issues).

Note that `pytest` distinguishes between failures and errors:
- A failure occurs if a test did not run successfully.
- An error occurs if an exception happened outside of the test function, for example inside a fixture.

## Test Directory Structure
```
tests/
  conftest.
  checks.py
  fixtures/
    Snakefile
    unit/
      config/
      data/
      create_fixtures.py
    integration/
      config/
      data/
      create_fixtures.py
  integration/
    test_functionality.py
    test_reproduce_Bos2016.py
    test_reproduce_Schuecker2015.py
    test_usage_examples.py
  meta/
    test_checks.py
  unit/
    test_input_output.py
    test_network.py
    test_aux_calcs.py
    test_meafield_calcs.py
```

`conftest.py` is a special `pytest` file, in which custom fixtures
and special `pytest` functions are defined. We, in particular, make use of the
`pytest_generate_tests` function, which considerably simplifies complex
parametrizations of tests.

`checks.py` is a collection of custom assert functions.

`fixtures/` contains all the data that is used for tests comparing real and
expected output of functions, as well as the files that creates the data
`create_fixtures.py` using the parameters defined in `config/`. The Snakefile
defines a workflow to create all fixtures at once.

`integration/` contains all integration tests.

`meta/` contains tests for custom assert functions.

`unit/` contains all unit tests.

## Test Design

Many test classes define the tested function as `staticmethod`, because the
function itself is not tightly related to class, but we still want to attach it
to the class for later reference. This allows us to call the function as an 'unbound function', without passing the instance to the function:
 `self.func()` = `func()` != `func(self)`.

There are two special fixtures that are definded in `conftest.py`:

If a test requires the `pos_keys` fixture, it will be parametrized such that
it tests all positive arguments the tested function (defined as a
`staticmethod` of the test class) takes. The list of all possible positive
arguments is defined within `conftest.py`.

If a test requires `output_test_fixtures`, pytest will pass the output fixtures
corresponding to the `output_key` defined as a test class variable. Those
output key results are checked into the repository for convenience, but can
be created from the sources (see Fixture Creation Workflow).
This allows us to parametrize the tests such that the function is tested in
different parameter regimes (e.g. mean-driven regime vs. fluctuation-driven
regime).

## Fixture Creation Workflow

Fixture creation is a sensible part of the testing framework as it supplies a
kind of ground truth to test against. Please make sure that your code is
trustworthy before running the fixture creation. Otherwise, tests might
incorrectly fail or pass.

The fixture creation workflow is defined using
[Snakemake](https://snakemake.readthedocs.io/en/stable/index.html), a workflow
management system using a Python based syntax. It is recommended to install it
in a separate conda environment (see
[Installation](https://snakemake.readthedocs.io/en/stable/getting_started/installation.html)).

To invoke the workflow and create the fixtures using the same conda environment
you are using the toolbox with, you first need to export the conda environment.
Therefore set `tests/fixtures/envs` as current working directory, activate the
corresponding conda environment and type
```
conda env export -f environment.yaml
```
Then open the created `environment.yaml` file, remove the last line starting
with `prefix` and add the line `- -e ../../../../` to the list at the end.
Change your current working directory to `tests/fixtures`, activate the conda
environment you have installed sakemake in and type
```
snakemake --use-conda --cores 1
```
The workflow then takes care of installing the necessary conda environments and
creating all fixtures that are specified within `tests/fixtures/config.yaml`.
By default, the workflow looks whether the requested fixtures exists and only
creates them if they don't.

It might be useful to first see what the workflow is planning to do by
triggering a 'dry-run' with: `snakemake -n`.
Furthermore the execution of single rules can be enforced with the `-R` flag,
e.g.: `snakemake --use-conda --cores 1 -R make_Bos2016_data`. This is useful
if one specific fixture should be re-created.

Have a look at the [Snakemake Documentation](https://snakemake.readthedocs.io/en/stable/index.html)
for more information.

# History of this Project

Mean-field theory is a tool that can help you understand the behaviour of your
network. Using this theory allows you to predict some features of a network
without running a single (often very time consuming) simulation.

At our institute, the
[INM-6 at the Research Center Juelich](https://www.fz-juelich.de/inm/inm-6/EN/Home/home_node_INM6.html),
we, among other things, investigate and develop such mean-field tools. Over the
years, more and more of the tools, developed by ourselves and other
researchers, have been implemented. In particular, the primary work for this
package has been done by Hannah Bos, Jannis Schuecker and Moritz Helias. Here
we extend the work published in the repository
[https://github.com/INM-6/neural_network_meanfield]
and make it available to a wider audience.

We restructured and rewrote the code with the aim to make it usable without
having to understand all details of the underlying theory. We simplified the
code. We introduced units and decided to store the parameters in separate .yaml
files. We wanted the users to only have to interact with one module. So, we
collected all the functionality in the `network.py` module. We ported the code
to Python 3. We made the whole thing a package. We expanded the documentation a
lot. We simplified saving results together with the parameters. And so on.

What we ended up with is the package that you are currently interested in. It
contains several tools for analyzing neuronal networks and makes them available
in a reusable format.
