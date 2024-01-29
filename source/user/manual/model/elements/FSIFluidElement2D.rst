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
|  The water is modeled as a linear inviscid, irrotational, and compressible fluid with hydrodynamic pressures p governed by the acoustic wave equation:	
.. math::
   \[{{\nabla }^{\text{2}}}p=\frac{1}{{{c}^{2}}}\frac{{{\partial }^{2}}p}{\partial {{t}^{2}}}\]   

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
