Functions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Functions to use the platform
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


To define test case
----------------------------------------------------

def test_cases ():
  if the number of the test case exists, return entrance_doors_coord, exit_doors_coord, list_direction, demand, num_layers
  else, return None
(entrance_doors_coord and exit_doors_coord are called inflows and outflows too)

To define the geometry of the domain
-------------------------------------------------

def create_geo():
  if the number of the geo_case exists, return domain, obstacles
  else, return None
  
To generate a rectangular finite volume grid
----------------------------------------------

def generate_mesh(domain, inflows, outflows, obstacles, num_layers):
  return mesh


See more in https://github.com/Ifsttar/PedSim/blob/master/docs/Use%20of%20the%20platform.rst 

Functions for boundaries conditions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To get time derivates of boundary conditions
----------------------------------------------

def time_derivative_bc(t, direction):
  if the number of the geo_case exists, return time_der
  else, return None
  

To compute density values in ghost cells
-------------------------------------------

def extrapolate_ghostcells(t, dt, rkstep, domain, density, demand, mesh, direction, layer):
  return density

It uses time_dericative_bc (t, direction).


Functions for flow equations and numerical schemes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To estimate speed thanks to the fundamental diagram
----------------------------------------------------

def estimate_speed(rho):
  return v_d
  
To compute flux
-----------------

def compute_flux(rho):
  return f
rho is a density matrix.

It uses estimate_speed (rho[np.logical_and(rho <= rho_j, rho > 0)])

Functions for WENO
-------------------------------------------

- To compute smoothness indicators for WENO
def smoothness_indicator(den):
  return beta
den is a column matrix in y direction and a row matrix in x direction.
beta is a matrix of the beta needed to compute WENO_scheme in one row or in one column.

- To build stencils for WENO
def construct_stencils(nx, ny):
  return xstencils, ystencils
x stencils is a matrix of the stencils needed to apply WENO-scheme in a row and y stencil is a matrix of the stencils needed to apply WENO-scheme in a column.
 
 - To reconstruct WENO
def WENO_scheme(density, stencils):
  return np.sum(denlAll * weightL, axis=0), np.sum(denrAll * weightR, axis=0)
It returns denl, denr wich are column matrix if we use WENO_scheme in y direction and row matrix if we use it in x direction.
density is a column matrix in y direction and a row matrix in x direction. stencils is a matrix of the stencils for a row or for a column. 
It uses smoothness_indicator (density).

To compute Lax_Friedrich scheme
---------------------------------

def lax_friedrich_flux(density, mesh, direction, phi_x, phi_y, xstencils, ystencils):
  return fij
fij is a matrix of all the flux at the interface between two cells in x direction or in y direction.
It uses WENO_scheme(density, stencils) for each row if the direction == x or for each column if direction == y.
It uses compute_flux (denl), compute_flux (denr)

Function for the dimensional splitting
---------------------------------------

def dimensional_splitting(density, F_i_j, dt, mesh, direction):
  return density
  
See more in https://github.com/Ifsttar/PedSim/blob/master/docs/Flow%20equations%2C%20numerical%20scheme%20and%20boundary%20conditions.rst

Functions for computing the direction
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To compute the fast marching scheme
-------------------------------------

def crowdedness_direction(nlayers, densities, layer):
    return gradpsi
It returns a gradient for a local direction term.

def fast_marching_scheme(density, mesh, layer, nlayers, densities):
  return -gradphi[1], -gradphi[0]
 
First, we compute the fast marching method thanks to the python module skfmm wich can calculate travel times. Then, we return the gradient of travel time and we add the one returns by the crowdedness_direction function. 
It uses estimate_speed(density) and crowdedness_direction (nlayers, densities, layer).
It returns gradient wich contains local and global direction terms. 

To compute the direction vectors
---------------------------------
 
def compute_direction_vectors(density, mesh, theta, layer, nlayers, densities):
  return phi_x, phi_y
It uses fast_marching_scheme (density, mesh, layer, nlayers, densities)

See more in https://github.com/Ifsttar/PedSim/blob/master/docs/Global%20and%20local%20direction.rst
  
General loop
~~~~~~~~~~~~~~

Time integration loop
------------------------------

def time_integration(domain, mesh, demands, directions, nlayers):
  return alldensities
It uses construct_stencils (nx, ny). 
It uses lax_friedrich_flux (density, mesh, dimension, phi_x, phi_y, xstencils, ystencils) for each direction, for the three TVD's steps for each time step.
It uses dimensionnal_splitting (density, fij, dt, mesh, dimension), compute_direction_vectors(sumdensities, mesh, theta, layer, nlayers, densities)
  
Main function
-----------------------------------

def main():

It uses test_cases (), create_geo (), generate_mesh (domain, inflows, outflows, obstacles, num_layers), time_integration(domain, mesh, demand, directions, num_layers), plot_solution(domain, solution, num_layers)

See more in https://github.com/Ifsttar/PedSim/blob/master/docs/General%20loop.rst 
  
To plot solution
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To plot snapshots of solution
-------------------------------

def plot_solution(domain, solution, nlayers):
