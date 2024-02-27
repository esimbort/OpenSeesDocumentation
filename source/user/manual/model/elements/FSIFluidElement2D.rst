.. _FSIFluidBoundaryElement2D:

FSIFluidBoundaryElement2D Element
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This command is used to construct an FSIFluidBoundaryElement2D element object. The FSIFluidBoundaryElement2D element is a 2-node linear acoustic boundary element object with the following features:

#. It is based on Eulerian pressure formulation [ZienkiewiczEtAl1978]_ , [ZienkiewiczEtAl2000]_ , [LøkkeEtAl2017]_ , for (Class I) fluid-structure interaction problem.
#. It uses a 2 inetgration points Gauss quadrature.
#. Depending on the input variables, it enables the implementation of radiation boundary, reservoir bottom absorption or free surface effects.

.. function:: element FSIFluidBoundaryElement2D $eleTag $n1 $n2 $cc $alpha $g <-thickness $thickness>

.. csv-table:: 
   :header: "Argument", "Type", "Description"
   :widths: 10, 10, 40

   $eleTag, |integer|, unique integer tag identifying element object
   $n1 $n2, 2 |integer|, the two nodes defining the element (-ndm 2 -ndf 1)
   $cc, |float|, speed of pressure waves in water
   $alpha, |float|, reservoir bottom reflection coefficient [LøkkeEtAl2017]_
   $g, |float|, acceleration due to gravity
   Optional:
   $thickness, |float|, the thickness in 2D problems (default 1.0).

.. figure:: figures/FSI_FE/FSIFluidBoundaryElement2D_geometry.png
	:align: center
	:figclass: align-center
	:width: 40%

	Nodes, Gauss points, local coordinate system
	
Theory
^^^^^^ 
Fluid behavior. Wave equation
-----------------------------
|  The water is modeled as a linear inviscid, irrotational, and compressible fluid with hydrodynamic pressures p governed by the Helmholtz acoustic wave equation:	

.. figure:: figures/FSI_FE/Helmholtz.png
	:align: center
	:figclass: align-center	
	:width: 10%
   
|  where
.. figure:: figures/FSI_FE/WaveVel.png
	:align: center
	:figclass: align-center	
	:width: 5%
|  denotes the speed of sound in the fluid, K and \rho are the bulk modulus and the mass density of the fluid, respectively.

Boundary conditions. Coupling and Radiation
-------------------------------------------
| We focus on the fluid-structure interaction (Class I problem), according to [ZienkiewiczEtAl2000]_ .
.. figure:: figures/FSI_FE/FSIProblem_geometry.png
	:align: center
	:figclass: align-center	
	:width: 40%
| The appropriate boundary conditions can now be imposed and linkage with the structural equations achieved. Therefore,
| On boundary 1: "Solid boundary"
.. figure:: figures/FSI_FE/BC1a.png
	:align: center
	:figclass: align-center	
	:width: 20%
| which leads to
.. figure:: figures/FSI_FE/BC1b.png
	:align: center
	:figclass: align-center	
	:width: 20%

| On boundary 2: "Solid boundary"
.. figure:: figures/FSI_FE/BC2a.png
	:align: center
	:figclass: align-center	
	:width: 20%
| leading to
.. figure:: figures/FSI_FE/BC2b.png
	:align: center
	:figclass: align-center	
	:width: 20%
| On boundary 3: "Free surface boundary"
| On the free surface the selected assumption is :math:`p=\rho g\eta`, which accounts for surface gravity waves, where :math:`\eta` is the elevation relative to the surface mean surface and :math:`g` is the acceleration due to gravity.
| This assumptions leads to the linearized free surface wave condition:
.. figure:: figures/FSI_FE/BC3a.png
	:align: center
	:figclass: align-center	
	:width: 15%
| On boundary 4: "Radiation boundary"
| The solution of the wave equation is composed of outgoing waves only: :math:`p = f(x - ct)`. Thus
.. figure:: figures/FSI_FE/BC4a.png
	:align: center
	:figclass: align-center	
	:width: 9%
| and
.. figure:: figures/FSI_FE/BC4b.png
	:align: center
	:figclass: align-center	
	:width: 9%
| By eliminating :math:`f'` we get 
.. figure:: figures/FSI_FE/BC4c.png
	:align: center
	:figclass: align-center	
	:width: 9%
| This relation is known as the Sommerfeld radiation condition. 
| The wave equation is to be solved in a volume :math:`\Omega_F`, subject to boundary conditions on its surface :math:`\Gamma_n`, leading to the following strong form for the fluid:
.. figure:: figures/FSI_FE/S_form.png
	:align: center
	:figclass: align-center	
	:width: 17%
| After multiplication by a weight function, integration by parts, application of the divergence theorem and susbstitution of BCs the weak form is shown below:
.. figure:: figures/FSI_FE/W_form.png
	:align: center
	:figclass: align-center	
	:width: 75%
| Standard Galerkin discretization applied to the weak form leads to
.. figure:: figures/FSI_FE/M_form.png
	:align: center
	:figclass: align-center	
	:width: 75%  
| The acoustic element stiffness matrix:
.. figure:: figures/FSI_FE/Ke_f.png
	:align: center
	:figclass: align-center	
	:width: 17%  
   
| The acoustic element mass matrix:   
.. figure:: figures/FSI_FE/Me_f.png
	:align: center
	:figclass: align-center	
	:width: 17%  

.. admonition:: Example: Three cases of valid inputs are shown below: 1. Radiation boundary, 2. Reservoir bottom absorption and 3. Surface waves effects.

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
