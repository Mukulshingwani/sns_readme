
# EEL2010 Programming Assignment

This folder contains various files which provide various building blocks, these blocks are finally assembled in file `main.py` and `ops.py` to solve the problem statement.

This README is divided into following sections:

- [Authors](#authors)

- [Getting Started](#getting-started)

    - [Installing The Dependencies](#installing-the-dependencies)
    - [Running the code](#running-the-code)
    - [Testing Our Code](#testing-our-code)
   - [Why and How are the these tests performed](#why-and-how-are-these-tests-performed)

- [Overview of the Code](#a-brief-overview-of-the-code)    
[Note: This section is completely optional for you to read but it might save a lot of time while evaluating.]
   - [utils/misc_utils.py](#utilsmisc_utilspy)
   - [utils/metrics.py](#utilsmetricspy)
   - [utils/test_ops_utils.py](#utilstest_ops_utilspy)
   - [utils/ops_utils.py](#utilsops_utilspy)
   - [ops.py](#opspy)
   - [main.py](#mainpy)

## Authors
- Mukul Shingwani (B20AI023)
- Nakul Sharma (B20AI024)

## Getting Started
We outline below the steps required to test and run our code.

### Installing the dependencies
To install all the required packages, simply switch to this directory (the directory containing this README file) in terminal
and run the following command (please make sure that `pip` is up-to-date)
```
pip install -r requirements.txt
```
This will install/update the following libraries:

- `NumPy`: Required for fast numerical computation on arrays.

- `matplotlib`: Required for plotting the signals.

- `pytest`: Required in order to test the building blocks of our code with just a single command.


### Running the code
Simply run the `main.py` file; This can be done in multiple ways but the easiest way if to run the following command in terminal when in the project directory inside the terminal:
```
python main.py
```
[NOTE: Please replace `python` with `python3` when running on a linux distribution.]

Running this command will:
   - Print the information regarding the two signals
   - Print metrics (like Energy Difference of the recovered signals from original signal, KL Divergence, etc.)
   - Plot the graphs of x1[n] with x[n], of x2[n] with x[n] and of x1[n] with x2[n].


### Testing our code
Once all the dependencies are installed, testing our code is as easy as running the following command in the terminal.
```
pytest .
```
This command will test all the functions which are fundamental to this programming assignment.
To be precise, following functions are tested (each of which is implemented by us):
- `conv1d`: convolution function used to convolve two signals

- `zero_pad`: this function is used to pad the signal with zeros. This function exists because we need to pad the signal
   before convolution so that the length of signal before and after convolution is same.

- `discrete_fourier_transform`: evident from the name itself, this function function calculates the Discrete Fourier Transform
   of a given signal.

- `inverse_fourier_transform`: calculates the Inverse Fourier Transform of a given Discrete Fourier Transform.

- [Imp] The integrity of `inverse_fourier_transform` and `discrete_fourier_transform` functions is also tested, i.e., we check if we are able to recover a dummy signal from its Discrete Fourier Transform by applying Inverse Fourier Transform on it. Passing this test ensures that everything is working perfectly fine.

P.S.: All of these functions are defined in `utils/ops_utils.py`. More explanation about these functions is waiting in the next section(s).
And all the functions to test these functions are stored in `utils/test_ops_utils.py`


### Why and How are these tests performed ??
All the functions that we define in `utils/ops_utils.py` are used to further build the functions for
Denoising (`denoise`) and DeBlurring (`deblur`) in the file `ops.py` and then we use `deblur` and `denoise`
functions in `main.py` to actually solve the problem statement.

So it is evident that the functions defined in `utils/ops_utils.py` are the stepping stones for us. These are
the base functions on which everything is dependent for solving this problem.
And thus it is very important to test the implementation of these functions so that we can be assured that the
final results are accurate.

We test our implementation of these functions by comparing the results from our implementation with their `NumPy`
counterparts. For example, here is how we tested our implementation of `conv1d` function: We take two signals `a` and `b`,
then we convolve them using our `conv1d` functions and then we check if the result produced by our function matches with the result
that we get when we convolve `a` and `b` using `NumPy`'s `np.convolve` function.

P.S.: all of the functions to test our implementation are available in the file `utils/test_ops_utils.py`.



## A Brief Overview of the Code.

The code contains two components: utilities and primary scripts. All the utility functions are stored in `utils` folder and primary scripts are `ops.py` and `main.py`. We will briefly go over important portion of the files in this section.

### `utils/misc_utils.py`
This file contains a function which is used to plot two signals in a contrastive manner.

### `utils/metrics.py`
This file contains the metrics functions which we use to measure the closeness of the recovered signal and the original signal x[n].

### `utils/test_ops_utils.py`
This file contains the test cases for the functions defined in `ops/ops_utils.py` (see below). In addition to those, we also test the integrity of our functions `discrete_fourier_transform` and `inverse_fourier_transform` as explained in the last point of [this section](#testing-our-code).

### `utils/ops_utils.py`
This file contains the functions for performing Convolution, Discrete Fourier, Transform and Inverse Fourier Transform and two additional functions to support these. We will first talk about these additional functions before jumping to the main ones.

- `convert_to_np`:
      This function converts the given arguements (which must are python iterables) and converts them into `NumPy` arrays of the given data type. The default data type chosen is `complex64` for great precision and handling of complex numbers of large magnitude.

- `reflective_pad`: This function is used pad the input signal using reflective padding before convolving it with the denoising kernel. The padding is done so that the length of the input and resultant convolved signal are equal.

- `conv1d`: This function convolution operation such that the length of the input and resultant convolved signal are equal.

- `discrete_fourier_transform`: This function returns the `num_samples` (an integer which serves as an arguement which should be passed to it) samples from the Discrete Time Fourier Transform (DTFT) of the input signal.     
Here are some more details regarding this:
   - We know that the DTFT of the signal is periodic and is continuous in `omega` from 0 to 2pi. Therefore, there exits infinite values of this DTFT for `omega` ranging from 0 to 2pi. But since the digital computers are great at storing quantized data and also, the signals can be recovered from their samples provided the sampling is done at an appropriate frequency: we conclude that instead of storing DTFT we will store samples of it. In our implementation we store 1000 samples for each DTFT. Reference for this: [MIT OCW course 6.341: Lecture 15](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-341-discrete-time-signal-processing-fall-2005/lecture-notes/lec15.pdf).

   - The implementation is 'vectorized', i.e., we converted each operation into a vector-matrix multiplication for faster computation. Explaining that would take up a lot of space here and hence we do not include it for the sake of sticking to the point. However, we do have some rough work available to show the method :)

   - The special case of h[n]: The given blur kernel h[n] starts from n=-2 instead of n=0. We handle this by passing `indices_collide=False` while calculating DFT for h[n]; this `indices_collide=False`, put simply, changes the range of summation to start from -2 instead of 0.

- `inverse_fourier_transform`: This function calculates and returns the Inverse Fourier Transform of a given Discrete Fourier Transform.
    - Its implementation is also vectorized in a way similar to that of the `discrete_fourier_transform`.

    - Since we operate on samples of DTFT, the integration in the formula IFT gets converted to a summation over all the samples.


### `ops.py`
This file is just an abstraction over `utils/ops_utils.py` and uses the functions defined in it to actually perform the deblurring and denoising. We use a uniform kernel for performing denoising.

### `main.py`
This is our main script. It contains two data classes:
- `RecoveredSignal`: This class stores the recovered signal and some properties in it. The `Metrics` class is used to store various metrics in this class.
- `Metrics`: This class stores various metrics in it.

It also contains the following functions:
- `get_data`: reads, processes and return x[n], y[n], h[n] and `kernel_sizes`. The `kernel_sizes` variable is a list of integers each of which represents the size of the denoising kernel that we should try.

- `calc_metrics`: calculates and returns various metrics between the recovered signal and a reference signal (original signal x[n] for us).

And here's what we do in the main body:  

[ Method A: Denoising then  Deblurring  
Method B: Deblurring then Denoising ]

- We initialize two lists which will contain signals obtained using method A and B.
- Now for each size in `kernel_sizes`(this takes the name `denoise_kernel_sizes` inside the main block), we apply both methods and store the recovered signals obtained in both cases in their respective lists, which are, `recovered_signals_method_a` and `recovered_signals_method_b`.
- Then we find the best signal obtained from each method seperately. We define the best signal as the signal which as maximum cross-corelation with the given signal `x[n]`. We store these best signals in variables `best_signal_a` and `best_signal_b`.
- Then we print some statistics regarding these two best signals and plot them with the original `x[n]`. We also plot them together to see how similar they are.
