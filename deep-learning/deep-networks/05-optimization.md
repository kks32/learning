# Optimization
Optimization involves finding the parameter $\theta$ of a neural network that significantly reduces a cost function $J(\theta)$, which includes a performance measure evaluated on the entire training set as well as regularization terms. 

In ML, we care about some performance measure $P$, which is defined with respect to a training set and may also be intractable. We optimize $P$ indirectly by reducing a cost function $J(\theta)$ hoping that will improve $P$, which is different from pure optimization where minimizing $J$ is a goal in and of itself. Expectation across data-generating distribution is: $J^*(\theta) = \mathbb{E}_{(x, y)\sim p_{data}} L(f(x;\theta), y)$, where $L$ is the per example loss function. The goal of ML is to reduce the expected generalization error. This quantity is known as the risk. The training process based on minimizing the average training error is known as *empirical risk minimization*. Rather than optimizing the risk directly, we optimize the empirical risk and hope that the risk decreases significantly as well. Empirical risk minimization is prone to overfitting. 

Surrogate loss function acts as a proxy to the loss function we care about, such as the negative log-likelihood of the correct class is used as a surrogate for 0-1 loss function, which has no useful derivatives (derivatives are either zero or undefined everywhere). The negative log-likelihood allows the model to estimate the conditional probability of the classes given the input. If the model can do well, then it can pick the classes that yield the least classification error. 

The main difference between optimization in general and optimization as we use for training algorithms is that they do not usually halt at a local minimum. Instead, an ML algorithm usually minimizes a surrogate loss function but halts when a convergence criteria based on early stopping is satisfied. 

Another difference is the objective function usually decomposes as a sum over the training examples. Computing expectations exactly is very difficult because it requires evaluating the model over every example in the dataset. In practice, these expectations are computed by randomly sampling a small number of examples from the dataset and taking the average over those samples. The standard error of mean is $\sigma/\sqrt{n}$, where $\sigma$ is the true standard deviation. Consider two sample sizes of 100 and 10,000, the 10,000 sample size requires 100 times more computation than the sample size of 100, but it only reduces the standard error by a factor of 10. 

A statistical estimation of the gradient is also beneficial as, in practice, there may be redundant examples in the training set or examples that make very similar contributions to the gradient. "Batch gradient" means an algorithm that uses the *full training set*. The size of examples used in stochastic methods depend on:
- Large batch sizes are more accurate but with less than linear returns
- Multicore architectures are less effectively utilized by small batch sizes
- The amount of memory required scales with the batch size and limits the batch sizes
- Smaller batch sizes add some regularization effect by adding noise to the learning process. This requires a smaller learning rater and hence is slow.

Some methods are sensitive to sampling effects. _Methods that compute the gradient $g$ are usually relatively robust and can handle small batches like 100. Second-order methods that compute the Hessian matrix $H^{-1}g$ require larger batch sizes like 10,000 to minimize fluctuations in the computation of $H^{-1}g$_. 

Minibatches are computed based on random sampling and require the samples to be independent. Two subsequent gradient estimates, and therefore, two subsequent mini-batches must also be independent of each other. Many examples in the training set are arranged such that they are highly correlated, e.g., a dataset of blood samples taken at different times for different patients arranged by the patient order. Shuffling these examples usually reduces this error. _Failing to shuffle the examples in any way can seriously reduce the effectiveness of any algorithm_.

SGD minimizes the generalization error. Most implementation of minibatch SGD shuffles the dataset once and then pass through it multiple times. On the first pass, each minibatch computes an unbiased estimate of the true generalization error. The estimates become biased in the second and subsequent passes as it is formed by resampling rather than new fair samples. Minibatch computes: $\hat{g} = \frac{1}{m} \nabla_\theta \sum_i L(f(x^{(i)};\theta), y^{(i)})$. Updating $\theta$ in the direction of $\hat{g}$ performs SGD on the generalization error. This only applies when the examples are not reused. In practice with the large growing dataset, each training example is used only once or even making incomplete passes through the training set. When using an extremely large training set, overfitting is no longer an issue. 

## Challenges in neural network optimization
Optimization in the neural network involves designing objective functions that are convex. Even convex functions have challenges when training neural nets. 

### Illconditioning
All condition can manifest by causing SGD to get "stuck" in the sense that even very small steps increase the cost function. The second-order Taylor series of the cost function predicts a gradient descent step of $\epsilon g$ will add $\frac{1}{2}\epsilon^2 g^T Hg - \epsilon g^T g$ to the cost function. Illconditioning of the gradient causes a problem when $\frac{1}{2}\epsilon^2 g^T Hg$ term is larger than $\epsilon g^T g$. In many cases, the gradient norm does not shrink significantly throughout the learning, but $\epsilon g^T g$ term grows by more than an order of magnitude. The learning becomes very slow despite steep gradients because the learning rater must shrink to compensate for the drastic changes in the curvature. 

### Local minima
Some convex functions have a flat region at the bottom rather than a single global minimum point. Any point on the flat region is an acceptable solution. DL models have a large number of local minima and can have higher costs than the global minimum. In practice, however, with sufficiently large neural networks, most local minima have a low-cost function value. It is important to find the true local minima rather than a point in parameter space that has a low but not minimal cost. A test that can rule out local minima is to plot the gradient with time. If the norm of gradient does not shrink to insignificant size, the problem is neither local minima nor any kind of critical points. 

### Plateaus, saddle points, and other flat regions
We can think of saddle points as local minima along one cross-section and a local maximum along another cross-section of the cost function. In low dimensions, the local minima are common. In high dimensions, local minima are rare, and saddle points are more common. 

Gradient descent is designed to move "downhill" and is not explicitly designed to seek a critical point. However, Newton's method is designed to solve for a point where the gradient is zero without modification. It can jump to a saddle point. 

### Cliffs and exploding gradients:
Neural networks with many layers have extremely steep gradients, resembling cliffs. Gradients may move fast and usually jump off the cliff structure. Cliffs are more common in Recurrent Neural Networks (RNNS) as it involves multiplication of many factors. 

### Long term dependencies
Repeated application of the same parameters, like in RNNs, gives rise to especially pronounced difficulties. Suppose we multiply by a matrix $W$ at each step; after $t$ steps, it is equivalent to $W^t$. If $W$ has an eigendecomposition of $= V diag(\lambda) V^{-1}$, then $W^t = (V diag(\lambda) V^{-1})^t = V diag(\lambda)^t V^{-1}$. Any eigenvalues $\lambda_i$ which are not near an absolute value of one, will either explode if they are greater than one or vanish if they are less than one. Vanishing gradients make it difficult to know which direction the parameter should move to improve the cost function. Deep feedforward networks avoid multiplying by $W$ so avoids the vanishing and exploding gradient problem, while RNNs do not. 

### Inexact gradients
Nearly all DL algorithms depend on the gradients' sampling-based estimates, such as using mini-batch in computing gradients of the training example. When the objective function is intractable, its gradient is also intractable. Neural networks are designed to account for imperfections in gradient estimates. One can avoid this issue by choosing a surrogate loss function than the true loss function. 

Much of the research is focused on difficulties of optimization when training arrives at a global minimum, local minimum, or saddle points. Neural networks do not often arrive at a region of smaller gradients. Indeed such critical points do not necessarily exist, e.g., loss function $- \log p(y|x; \theta)$ can lack a global minimum point and instead asymptotically approach some value as the model becomes more confident. In low dimensions, starting on the wrong side of the cost function hill, the algorithm may get stuck on this wrong side. The algorithm could potentially circumnavigate the hill in high dimensions, but it results in longer training trajectories and excessive training. 

Gradient descent and other learning algorithms usually make small moves. The gradient of the objective function is computed approximately by sampling with bias or variance in the estimate of the correct direction. The objective function may suffer from poor conditioning, causing the region where the gradient provides a good model of the objective function to be very small. In these cases, local descent with a step size of $\epsilon$ may define a reasonably short path to the solution, but we can only compute the local descent direction with step size $\delta << \epsilon$. In these cases, although local descent may define a path to the solution, the path contains many steps and incurs high computation cost. Local descent could lead to moving in the wrong direction or a larger number of steps required to reach the solution, which of these is a more relevant problem that makes optimization of neural network difficult is still an open problem. Hence, choosing good initial points for the optimization algorithm is essential.

## Stochastic Gradient Descent

Previously we used a fixed learning rate $\epsilon$. In practice, it is necessary to decrease the learning rate over time $\epsilon_k$ with iteration $k$. SGD gradient estimator introduces a source of noise from random sampling that does not vanish even when we arrive at a minimum. While the batch gradient descent becomes zero as a result of using the true cost function, hence can use a fixed learning rate. It is common to decay the learning rater linearly until iteration $\tau$ after $\tau$ leaves $epsilon$ constant. 
$$\epsilon_k = (1-\alpha) \epsilon_0 + \alpha \epsilon_\tau \qquad \alpha = k / \tau$$. 
Typically, $\tau$ is set to a number of iterations required to make a few hundred passes through the training set. $\epsilon_\tau$ is roughly 1% of the value of $\epsilon_0$. If the learning rate $\epsilon_0$ is too low, the learning proceeds slowly, and a large $\epsilon_0$ can cause violent oscillations resulting in optimization getting stuck at high-cost functions. Typically, the optimal learning rate, in terms of total training time and final cost, is higher than the learning rate that yields the best performance after the first 100 iterations or so. The best approach is to monitor the first several iterations and use a learning rate higher than the best performing learning rate at this time, but not so high it causes severe instability. SGD computation time does not grow with increasing training examples. For SGD, the excess error in convergence is $O(\frac{1}{sqrt{k}})$ after $k$ iterations. Faster convergence may mean overfitting.

## Momentum

Momentum is designed to accelerate learning, especially in the face of high curvature, small but consistent gradients, or noisy gradient. Momentum accumulates an exponentially decaying moving average of past gradient and continues to move in their direction. Momentum introduces a variable $v$ that plays the role of velocity - direction and speed at which the parameters move through the parameter space. In physics, momentum is defined as mass times velocity. In ML, we assume a unit mass, so the particle velocity is its momentum. 
$$ v \leftarrow \alpha v - \epsilon g$$
$$\theta \leftarrow \theta + v$$
The gradient of the loss function $g$ is written as 
$$ g = \nabla_\theta \left(\frac{1}{m} \sum_{i=1}^m L(f((x^{(i)}; \theta), y^{(i)})\right)$$

The larger the $\alpha$ is relative to $\epsilon$, the more previous gradients affect the current direction. The size of the step depends on how large and how aligned a sequence of gradients are. The step size is largest when many successive gradient points in exactly the same direction. Momentum hyperparameters can be thought of in terms of $\frac{\epsilon ||g||}{1 - \alpha}$. So an $\alpha = 0.9$ corresponds to multiplying the maximum speed by ten relative to gradient descent algorithms. Common values of $\alpha$ are 0.5, 0.9, and 0.99. The value of $\alpha$ can be adapted over time, starting with a small value and increasing with time. Momentum is similar to a hockey puck descending down an icy slope, gathering speed in the direction of the steepest part of the surface. 


## Nestrov momentum

The gradient is evaluated after the current velocity is applied. In SGD, Nestrov momentum does not improve the rate of convergence. The gradient $g$ is computed differently as:
$$ g = \nabla_\theta \left(\frac{1}{m} \sum_{i=1}^m L(f((x^{(i)}; \theta + \alpha v), y^{(i)})\right)$$

## Parameters initialization strategies

DL models are strongly affected by choice of initialization. Modern initialization strategies are simple and heuristic. It is usually best to initialize each unit to compute different functions from all other units and motivates random initialization of parameters. We initialize the weights in the model randomly drawn from a Gaussian or uniform distribution. A larger initial weight yields a stronger symmetry-break effect, helping to avoid redundant units. Initial weights that are too large may explode or cause chaos in recurrent networks. SGDs with a small learning rate may not update the weights much from their initial values. When computationally possible, it is a good idea to treat the initial weights as hyperparameters. A good rule of thumb in choosing the initial scaling is to look at the range or standard deviation of activation or gradients in minibatch. If weights are too small, the range of activation across the minibatch will shrink as it propagates through the network. Identify the first layer with unacceptably small activation and increase its weights. Setting the bias to zero is a good initial value and is compatible with most weight initialization strategies. Non-zero biases are sometimes needed to avoid saturation of ReLU by setting it to 0.1 instead of 0. The precision parameter $\beta$ is typically set to 1.

## Algorithms with adaptive learning rate
Learning rate is one of the most challenging hyperparameters to set, and it significantly affects the model performance. The cost function is highly sensitive in some directions and insensitive in others. It is possible to separate the rate for different parameters. Early methods involved increasing the learning rate if the partial derivative remains the same sign and decreasing the learning rate if the partial derivative changes sign.

### AdaGrad
AdaGrad individually adapts the learning rate of all model parameters by scaling them inversely proportional to the square root of the sum of all historical squared values of the gradient. Large changes in the partial derivative mean a rapid decrease in the learning rate. Smaller changes in the partial derivative make a relatively small decrease in the learning rate and make significant progress in the more gently sloped parameter space direction.

### RMSprop
RMSprop modified the accumulated gradient into an exponentially weighted moving average. RMSprop disregards history from the extreme past so it can converge rapidly after finding a convex bowl. A new hyperparameter $\rho$ controls the length of the moving average. 

### Adam
**Ada**ptive **M**oment is a combination of RMSprop and momentum with a few important distinctions. First, momentum is incorporated directly as the estimate of the first-order moment with an exponential weighting of the gradient. It also includes a bias correction to estimates of both first-order moments (the momentum term) and uncentered second-order moment to account for their initialization at the origin. Adam is robust to the choice of hyperparameters, though learning rates sometimes need to be changed from the suggested default. The default values are: step size $\epsilon = 0.001$, decay rates $\rho_1$ and $\rho_2$ must be within $[0, 1]$ and have a default values of 0.9 and 0.999 respectively, and $\delta = 1E-8$.

### Newton's method
Newton's method is an optimization scheme based on a second-order Taylor series expansion to approximate $J(\theta)$ near $\theta_0$:

$$J(\theta) \approx J(\theta_0) + (\theta - \theta_0)^T \nabla_\theta J(\theta_0) + 0.5 (\theta - \theta_0)^T H (\theta - \theta_0)$$

Newton update is: $\theta^* = \theta_0 - H^{-1} \nabla_\theta J(\theta_0)$., where $H$ is the Hessian of $J$. For surfaces not quadratic, as long as $H$ is positive definite, NEwton's method can be applied iteratively. If the eigenvalues of the $H$ are not all positives (such as at a saddle point), then Newton's method can move in the wrong direction. Regularized Hessian avoids this: $\theta^* = \theta_0 - [H (f(\theta_0)) + \alpha I]^{-1} \nabla_\theta f(\theta_0)$. The application of Newton's method in deep learning is limited due to its significant computational cost of calculating the inverse of H at $O(n^3)$.

### Conjugate-Gradient
Conjugate-Gradient avoids computing the inverse of the Hessian by iteratively descending in teh conjugate directions. We seek to find a search direction that is conjugate to previous search direction and does not undo the progress made in that direction. At training iteration $t$, the next search direction $d_t$ is computed as: $d_t = \nabla_\theta J(\theta) + \beta_t d_{t -1}$, where $\beta_t$ controls how much of the direction $d_{t - 1}$ we should add back to the current search direction. Two directions $d_t$ and $d_{t-1}$ are conjecate if $d_{t}^T H d_{t-1} = 0$. Conjugacy is imposed by calculating the eigen vectors of $H$ to choose $\beta_t$. 

### Broyden-Fletcher-Goldfard-Shanno (BGFS)
BFGS is similar to conjugate gradient, while conjugate gradient takes a direct approach to approximate the $H^{-1}$ in Newton's update. BFGS approximates the inverse with matrix $M_t$ iteratively refined by low-rank updates to become a better approximation of $H^{-1}$. Once approximate $M_t$ is updated, the direction of descent $\rho_t$ is determined by $\rho_t = M_t g_t$. A line search is performed in this direction with step $\epsilon^*$: $\theta_{t+1} = \theta_t + \epsilon^* \rho_t$. BFGS needs less time on line search but needs to store the approximate inverse Hessian matrix $M$ that requires $O(n^2)$ memory. 

### Limited-Memory BFGS (L-BFGS)
L-BFGS avoids storing the complete inverse approximate $M$, by assuming $M^{(t-1)}$ is the identity matrix, rather than storing the approximate from one step to next. L-BFGS is well behaved even when the linear search is only approximate. The memory cost is $O(n)$ per step. 

## Other optimization strategies
### Batch normalization
Batch normalization is the reparameterization of a DL network and reduces the problem of coordinating updates across multiple layers. Let $H$ be a minibatch activation of the layer to normalized, arranged by a design matrix with the activation of each example appearing in a row of the matrix. To normalize $H$, $H^\prime = \frac{H - \mu}{\sigma}$, $\mu$ is vector of mean of each unit $\mu = \frac{1}{m}\sum_i H_i$ and $\sigma$ is the standard deviation $\sigma = \sqrt{\delta + \frac{1}{m} \sum_i (H - \mu_i)^2}$, $\delta =1E-8$. We backpropagate through these operations for computing mean and standard deviations and applying them to normalize $H$. This means the gradient will never act to increase the standard deviation or mean of $h_i$; normalization removes such action and zeros the components of those gradients. Due to normalization, lower layers do not cause any harmful effects and no longer have any beneficial effects either. 

### Coordinate descent
Coordinate descent solves the optimization problem by breaking it into separate pieces. If we minimize $f(x)$ with respect to a single variable $x_i$, then minimizing to another $x_j$ and so on, repeatedly cycling through all variables, we are guaranteed to arrive at a (local) minimum. Block coordinate descent refers to minimizing for a subset of variables simultaneously. Coordinate descent makes sense when different variables in optimization can be clearly separated into groups that play relatively isolated roles. The algorithm is not a very good strategy when one variable's value strongly influences another's optimal value. 

### Poylok averaging
The algorithm involves averaging several points in a trajectory through parameter space visited by the optimization algorithm. If $t$ iteration of gradient descent visits the points $\theta^{(1)}, \dots \theta^{(t)}$, then the output of Poylok averaging is $\hat{\theta}^{(t)} = \frac{1}{t}\sum_i \theta^{(i)}$. On gradient descent, this approach has a strong convergence guarantee and performs well practically. In a neural network, its justification is more heuristic. In nonconvex optimization, an exponential decay function is used. 

## Supervised pretraining
Directly training a model to solve a specific task can be too ambitious if the model is too complex and hard to optimize. It is sometimes more efficient to train a simpler model to solve the task and then make the model more complex. Training simple models on simple tasks before confronting the challenges of training the desired model to perform the desired task is collectively known as *pretraining*.

**Greedy algorithms** break a problem into many components and then solve each component's optimal version in isolation. Unfortunately, combining individually optimized components is not guaranteed to yield a completely optimal solution. Greedy algorithms can be followed by _fine tuning_ to search for the optimal solution to full problems. 

In practice, _it is more important to choose a model family that is easy to optimize than to use a powerful training algorithm_. Linear models are easy to train as they linearly increase in only one direction. Even if linear models are way off the correct value, computing its gradient informs the direction to move to reduce the cost function. Hence, LSTM, ReLU, and max outs are all moving towards more linear functions. 

**Continuation methods** make optimization easier by choosing initial points to ensure the local optimization spends most of its time in a well-behaved region of space. Continuation methods construct a series of objective functions over the same parameter space. To minimize cost function $J(\theta)$, we construct new cost functions ${J^{(0)}, \dots J^{(n)}}$. The cost functions are designed to be increasingly difficult. $J^{(i)}$ is easier than $J^{(i+1)}$, by which we mean it is well-behaved over more of a $\theta$ space. The series of cost functions are designed so that a solution to one is a good initial value to the next. Continual methods deal with local minima, but local minima are not believed to be a problem in neural networks. The continuation method is still useful in improving the local update direction and progress towards a global solution.