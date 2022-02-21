
Syntax comparison
=================

In order to compare the syntax between different API's, let's initialize the following problem in the different API's:

.. math::

    & \min \;\; \sum_{i,j} 2 x_{i,j} \; y_{i,j} \\
    s.t. & \\
    & x_{i,j} - y_{i,j} \; \ge \; i \qquad \forall \; i,j \in \{1,...,N\} \\
    & x_{i,j} + y_{i,j} \; \ge \; 0 \qquad \forall \; i,j \in \{1,...,N\}





In `JuMP` the formulation translates to the following code:

 .. code-block:: julia

    using JuMP

    function create_model(N)
        m = Model()
        @variable(m, x[1:N, 1:N])
        @variable(m, y[1:N, 1:N])
        @constraint(m, [i=1:N, j=1:N], x[i, j] - y[i, j] >= i)
        @constraint(m, [i=1:N, j=1:N], x[i, j] + y[i, j] >= 0)
        @objective(m, Min, sum(2 * x[i, j] + y[i, j] for i in 1:N, j in 1:N))
        return m
    end

The same model in 'linopy' is initialized by

 .. code-block:: python

     from linopy import Model
     from numpy import arange


     def create_model(N):
         m = Model()
         coords = [arange(N), arange(N)]
         x = m.add_variables(coords=coords)
         y = m.add_variables(coords=coords)
         m.add_constraints(x - y >= arange(N))
         m.add_constraints(x + y >= 0)
         m.add_constraints(x + y >= 0)
         m.add_objective((2 * x).sum() + y.sum())
         return

Note that the syntax is quiet similar. An important difference lays in the fact that `linopy` operates all arithmetic operations on **variable arrays**, while the JuMP syntax uses control variables `i` and `j`.

In `Pyomo` the code would look like

 .. code-block:: python

     from common import profile
     from numpy import arange
     from pyomo.environ import ConcreteModel, Constraint, Objective, Set, Var
     from pyomo.opt import SolverFactory


     def model(n, solver, integerlabels):
         m = ConcreteModel()
         if integerlabels:
             m.i = Set(initialize=arange(n))
             m.j = Set(initialize=arange(n))
         else:
             m.i = Set(initialize=arange(n).astype(float))
             m.j = Set(initialize=arange(n).astype(str))

         m.x = Var(m.i, m.j, bounds=(None, None))
         m.y = Var(m.i, m.j, bounds=(None, None))

         def bound1(m, i, j):
             return m.x[(i, j)] - m.y[(i, j)] >= i

         def bound2(m, i, j):
             return m.x[(i, j)] + m.y[(i, j)] >= 0

         def objective(m):
             return sum(2 * m.x[(i, j)] + m.y[(i, j)] for i in m.i for j in m.j)

         m.con1 = Constraint(m.i, m.j, rule=bound1)
         m.con2 = Constraint(m.i, m.j, rule=bound2)
         m.obj = Objective(rule=objective)
         return m

which is heavily based on the internal call of functions in order to define the constraints.