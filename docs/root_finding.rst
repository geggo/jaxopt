.. _root_finding:

Root finding
============

This section is concerned with root finding, that is finding :math:`x` such
that :math:`F(x, \theta) = 0`.

Bisection
---------

.. autosummary::
  :toctree: _autosummary

    jaxopt.Bisection

Bisection is a suitable algorithm when :math:`F(x, \theta)` is one-dimensional
in :math:`x`.

Instantiating and running the solver
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

First, let us consider the case :math:`F(x)`, i.e., without extra argument
:math:`\theta`.  The ``Bisection`` class requires a bracketing interval
:math:`[\text{lower}, \text{upper}]`` such that :math:`F(\text{lower})` and
:math:`F(\text{upper})` have opposite signs, meaning that a root is contained
in this interval as long as :math:`F` is continuous.  For instance, suppose
that we want to find the root of :math:`F(x) = x^3 - x - 2`. We have
:math:`F(1) = -2` and :math:`F(2) = 4`. Since the function is continuous, there
must be a :math:`x` between -2 and 4 such that :math:`F(x) = 0`::

  from jaxopt import Bisection

  def F(x):
    return x ** 3 - x - 2

  bisec = Bisection(optimality_fun=F, lower=1, upper=2)
  print(bisec.run().params)

``Bisection`` successfully finds the root ``x = 1.521``.
Notice that ``Bisection`` does not require an initialization,
since the bracketing interval is sufficient.

Differentiation
~~~~~~~~~~~~~~~

Now, let us consider the case :math:`F(x, \theta)`.  For instance, suppose that
``F`` takes an additional argument ``factor``.  We can easily differentiate
with respect to ``factor``::

  def F(x, factor):
    return factor * x ** 3 - x - 2

  def root(factor):
    bisec = Bisection(optimality_fun=F, lower=1, upper=2)
    return bisec.run(factor=factor).params

  # Derivative of root with respect to factor at 2.0.
  print(jax.grad(root)(2.0))

Under the hood, we use the implicit function theorem in order to differentiate the root.
See the :ref:`implicit differentiation <implicit_diff>` section for more details.

Scipy wrapper
-------------

.. autosummary::
  :toctree: _autosummary

    jaxopt.ScipyRootFinding


Broyden's method
--------------

.. autosummary::
  :toctree: _autosummary

    jaxopt.Broyden

Broyden's method is an iterative algorithm suitable for nonlinear root equations in any dimension.
It is a quasi-Newton method (like L-BFGS), meaning that it uses an approximation of the Jacobian matrix
at each iteration.
The approximation is updated at each iteration with a rank-one update.
This makes the approximation easy to invert using the Sherman-Morrison formula, given it does not use too many
updates.
One can control the number of updates with the ``history_size`` argument.
Furthermore, Broyden's method uses a line search to ensure the rank-one updates are stable.

Example::

    import jax.numpy as jnp
    from jaxopt import Broyden

    def F(x):
      return x ** 3 - x - 2

    broyden = Broyden(fun=F)
    print(broyden.run(jnp.array(1.0)).params)


For implicit differentiation::

    import jax
    import jax.numpy as jnp
    from jaxopt import Broyden

    def F(x, factor):
      return factor * x ** 3 - x - 2

    def root(factor):
      broyden = Broyden(fun=F)
      return broyden.run(jnp.array(1.0), factor=factor).params

    # Derivative of root with respect to factor at 2.0.
    print(jax.grad(root)(2.0))
