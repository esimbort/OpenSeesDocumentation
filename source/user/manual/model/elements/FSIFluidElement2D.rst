.. _FSIFluidElement2D:

FSIFluidElement2D Element
^^^^^^^^^^^^^^^^^^^^^^^^^

This command is used to construct an FSIFluidElement2D element object. The FSIFluidElement2D element is a 4-node bilinear acoustic element with the following features:

#. It is based on Eulerian pressure formulation [ZienkiewiczEtAl1978]_ , [ZienkiewiczEtAl2000]_ , [LøkkeEtAl2017]_ , for (Class I) fluid-structure interaction problem.
#. It uses a full 2x2 Gauss quadrature, so it has a total of 4 integration points.

.. function:: element FSIFluidElement2D $eleTag $n1 $n2 $n3 $n4 $cc

.. csv-table:: 
   :header: "Argument", "Type", "Description"
   :widths: 10, 10, 40

   $eleTag, |integer|, unique integer tag identifying element object
   $n1 $n2 $n3 $n4, 4 |integer|, the four nodes defining the element (-ndm 2 -ndf 1)
   $cc, |float|, speed of pressure waves in water


.. figure:: FSIFluidElement2D_geometry.png
	:align: center
	:figclass: align-center

	Nodes, Gauss points, local coordinate system

.. note::

   Valid queries to the FSIFluidElement2D element when creating an ElementRecorder object are:
   
   *  '**pressure**', '**pressureVel**', '**pressureAccel**':
       *  Added hydrodynamic pressure and its first and second time derivatives.
	   
Theory
^^^^^^ 
Fluid behavior. Wave equation
-----------------------------
|  The water is modeled as a linear inviscid, irrotational, and compressible fluid with hydrodynamic pressures p governed by the Helmholtz acoustic wave equation:	

.. math::
   \nabla^2 p = \frac{1}{c^2} \frac{\partial^2 p}{\partial t^2}
   
|  where
.. math::
   c = \sqrt{\frac{K}{\rho}}
|  denotes the speed of sound in the fluid, K and \rho are the bulk modulus and the mass density of the fluid, respectively.

Boundary conditions. Coupling and Radiation
-------------------------------------------
| We focus on the fluid-structure interaction (Class I problem), according to [ZienkiewiczEtAl2000]_ .
.. figure:: FSIProblem_geometry.png
	:align: center
	:figclass: align-center	
	:width: 40%
| The appropriate boundary conditions can now be imposed and linkage with the structural equations achieved. Therefore,
| On solid boundary 1:
.. math::

   {{\dot{\bar{u}}}_{{n_{F,\,h}}}} = \ddot{w}_{{n_{F,\,h}}}^{s} = \mathbf{n}_{F,\,h}^{\text{T}} \mathbf{\ddot{w}}^{s}
| which leads to
.. math::

   \rho \, \mathbf{n}_{F,\,h}^{\text{T}} \mathbf{\ddot{w}}^{s} = -\frac{\partial p}{\partial {n_{F,\,h}}}

| On solid boundary 2:
.. math::

   {{\dot{\bar{u}}}_{{n_{F,\,b}}}} = \ddot{w}_{{n_{F,\,b}}}^{s} = \mathbf{n}_{F,\,b}^{\text{T}} \mathbf{\ddot{w}}^{s}
| leading to
.. math::

   \rho \, \mathbf{n}_{F,\,b}^{\text{T}} \mathbf{\ddot{w}}^{s} + \frac{1}{c} \left( \frac{1-\alpha}{1-\alpha} \right) \dot{p} = -\frac{\partial p}{\partial {n_{F,\,b}}}
| On solid boundary 3:
| On the free surface, we can take either :math:`p=0` or :math:`p=\rho g\eta`. The latter accounts for surface gravity waves, where :math:`\eta` is the elevation relative to the surface mean surface and :math: `g` is the acceleration due to gravity.




| The wave equation is to be solved in a volume \Omega_F, subject to boundary conditions on its surface \Gamma_n, leading to the follwing strong form for the fluid:
.. math::

   \left( \text{S} \right)\quad \left\{ \begin{array}{ll}
   \nabla^2 p = \frac{1}{c^2} \ddot{p} & \text{in } \Omega \\
   \frac{\partial p}{\partial n} = -\rho \dot{u}_n & \text{on } \Gamma_n \\
   \end{array} \right.
   
| After multiplication by a weight function, integration by parts, application of the divergence theorem and susbstitution of BCs the weak form is shown below:
.. math::

   \left( \text{W} \right)\left\{ \int_{\Omega_F} \delta p \left( {\left( \nabla \right)}^{\text{T}} \nabla p + \frac{1}{{c}^{2}} \ddot{p} \right) d\Omega + \rho \int_{\Gamma_1} \delta p \dot{u}_{n_{F,h}} d\Gamma + \rho \int_{\Gamma_2} \delta p \dot{u}_{n_{F,b}} d\Gamma + \frac{1}{c} \left( \frac{1 - \alpha}{1 + \alpha} \right) \int_{\Gamma_2} \delta p \dot{p} d\Gamma + \frac{1}{g} \int_{\Gamma_3} \delta p \ddot{p} d\Gamma + \frac{1}{c} \int_{\Gamma_4} \delta p \dot{p} d\Gamma = 0 \ \ (4.14) \right.
| Standard Galerkin discretization applied to the weak form leads to
.. math::

   \left( \text{M} \right)\left\{
   \begin{align*}
      & \sum_{e}^{n_{el}} \delta \mathbf{P}_{e}^{\text{T}} \underbrace{\left( \int_{\Omega^{e}} \left( {\left( \nabla \mathbf{N}_{F} \right)}^{\text{T}} \nabla \mathbf{N}_{F} \right) d\Omega \right)}_{\mathbf{K}_{F}^{e}} \mathbf{P}_{e} \\
      & + \sum_{e}^{n_{el}} \delta \mathbf{P}_{e}^{\text{T}} \underbrace{\left( \frac{1}{c^2} \int_{\Omega^{e}} \mathbf{N}_{F}^{\text{T}} \mathbf{N}_{F} d\Omega \right)}_{\mathbf{M}_{F}^{e}} {\mathbf{\ddot{P}}}_{e} \\
      & + \sum_{e}^{n_{el}} \delta \mathbf{P}_{e}^{\text{T}} \underbrace{\left( \rho \int_{\Gamma_{1}^{e}} \mathbf{N}_{F}^{\text{T}} {\dot{u}}_{n_{F,h}} d\Gamma \right)}_{\mathbf{R}_{F,h}^{e}} \\
      & + \sum_{e}^{n_{el}} \delta \mathbf{P}_{e}^{\text{T}} \underbrace{\left( \rho \int_{\Gamma_{2}^{e}} \mathbf{N}_{F}^{\text{T}} {\dot{u}}_{n_{F,b}} d\Gamma \right)}_{\mathbf{R}_{F,b}^{e}} \\
      & + \cdots \\
      & \cdots + \sum_{e}^{n_{el}} \delta \mathbf{P}_{e}^{\text{T}} \underbrace{\left( \frac{1}{c} \left( \frac{1-\alpha }{1+\alpha } \right) \int_{\Gamma_{2}^{e}} \mathbf{N}_{F}^{\text{T}} \mathbf{N}_{F} d\Gamma \right)}_{\mathbf{C}_{F,b}^{e}} {\mathbf{\dot{P}}}_{e} \\
      & + \sum_{e}^{n_{el}} \delta \mathbf{P}_{e}^{\text{T}} \underbrace{\left( \frac{1}{g} \int_{\Gamma_{3}^{e}} \mathbf{N}_{F}^{\text{T}} \mathbf{N}_{F} d\Gamma \right)}_{\mathbf{W}_{F}^{e}} {\mathbf{\ddot{P}}}_{e} \\
      & + \sum_{e}^{n_{el}} \delta \mathbf{P}_{e}^{\text{T}} \underbrace{\left( \frac{1}{c} \int_{\Gamma_{4}^{e}} \mathbf{N}_{F}^{\text{T}} \mathbf{N}_{F} d\Gamma \right)}_{\mathbf{C}_{F,r}^{e}} {\mathbf{\dot{P}}}_{e} = 0
   \end{align*}
   \right.\ 
   
| The element stiffness matrix:
.. math::

   \mathbf{K}_{F}^{e} = \int_{\Omega^{e}} {\left( \nabla \mathbf{N}_{F} \right)}^{\text{T}} \nabla \mathbf{N}_{F} \, d\Omega
   
| The element mass matrix:   
.. math::

   \mathbf{M}_{F}^{e} = \frac{1}{{c}^{2}} \int_{\Omega^{e}} \mathbf{N}_{F}^{\text{T}} \mathbf{N}_{F} \, d\Omega
   
.. admonition:: Example 

   1. **Tcl Code**

   .. code-block:: tcl

      # set up a 2D-1DOF model
      model Basic -ndm 2 -ndf 1
      node 1  0.0  0.0
      node 2  1.0  0.0
      node 3  1.0  1.0
      node 4  0.0  1.0
      
      # create the acoustic element with speed of pressure waves in water, c = 1.440000e+03
      set cc 1.440000e+03
      element FSIFluidElement2D  1  1 2 3 4  $cc
      
      # record added hydrodynamic pressures at element nodes (4 columns, 1 for each node)
      recorder Element  -xml  pressure_out.xml  -ele  1  pressure
      # record first time derivative of added hydrodynamic pressures at element nodes (4 columns, 1 for each node)
      recorder Element  -xml  pressureVel_out.xml  -ele  1  pressureVel

   2. **Python Code**

   .. code-block:: python

      # set up a 2D-1DOF model
      model('Basic', '-ndm', 2, '-ndf', 1)
      node(1, 0.0, 0.0)
      node(2, 1.0, 0.0)
      node(3, 1.0, 1.0)
      node(4, 0.0, 1.0)
      
      # create the acoustic element with speed of pressure waves in water, c = 1.440000e+03
      cc = 1.440000e+03
      element('FSIFluidElement2D', 1, 1,2,3,4, cc)
      
      # record added hydrodynamic pressures at element nodes (4 columns, 1 for each node)
      recorder('Element', '-xml', 'pressure_out.xml', '-ele', 1, 'pressure')
      # record first time derivative of added hydrodynamic pressures at element nodes (4 columns, 1 for each node)
      recorder('Element', '-xml', 'pressureVel_out.xml', '-ele', 1, 'pressureVel')

Code Developed by: **Massimo Petracca** at ASDEA Software, Italy.

.. [ZienkiewiczEtAl1978] | Zienkiewicz O.C., Bettess P. "Fluid-structure dynamic interaction and wave forces. An introduction to numerical treatment", Inter. J. Numer. Meth. Eng.., 13(1): 1–16. (`Link to article <https://onlinelibrary.wiley.com/doi/10.1002/nme.1620130102>`_)
.. [ZienkiewiczEtAl2000] | Zienkiewicz O.C., Taylor R.L. "The Finite Element Method", Butterworth-Heinemann, Vol.1, 5th Ed., Ch.19.
.. [LøkkeEtAl2017] Løkke A., Chopra A.K. "Direct finite element method for nonlinear analysis of semi-unbounded dam–water–foundation rock systems", Earthquake Engineering and Structural Dynamics 46(8): 1267–1285. (`Link to article <https://onlinelibrary.wiley.com/doi/abs/10.1002/eqe.2855>`_)
