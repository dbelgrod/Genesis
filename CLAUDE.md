# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Development Commands

### Testing
```bash
# Run tests (excluding benchmarks)
pytest -m "not benchmarks"

# Run specific test file
pytest tests/test_rigid_physics.py

# Run tests in parallel
pytest -n auto

# Run benchmarks separately
pytest -m benchmarks
```

### Code Formatting and Linting
```bash
# Format code with black (line length 120)
black genesis/ --line-length 120

# Check formatting without making changes
black genesis/ --check --line-length 120
```

### Installation
```bash
# Install in development mode with dev dependencies
pip install -e ".[dev]"

# Install render dependencies (for LuisaRender)
pip install -e ".[render]"

# Install docs dependencies
pip install -e ".[docs]"
```

### Running Examples
```bash
# Use the gs command-line tool
gs examples/rigid/single_franka.py

# Or run directly with Python
python examples/rigid/single_franka.py
```

## High-Level Architecture

Genesis is a universal physics engine and robotics simulation platform with the following key components:

### Core Structure

1. **Scene** (`genesis/engine/scene.py`): Central container for all simulation components
   - Manages the simulator, entities, and visualizer
   - Entry point for most user interactions
   - Controls simulation stepping and rendering

2. **Simulator** (`genesis/engine/simulator.py`): Manages multiple physics solvers
   - Coordinates between different solver types (rigid, MPM, SPH, FEM, PBD, etc.)
   - Handles inter-solver coupling through the Coupler
   - Manages simulation state and stepping

3. **Solvers** (`genesis/engine/solvers/`): Physics engines for different material types
   - **RigidSolver**: Rigid body and articulated body dynamics
   - **MPMSolver**: Material Point Method for deformable materials
   - **SPHSolver**: Smoothed Particle Hydrodynamics for fluids
   - **FEMSolver**: Finite Element Method for deformables
   - **PBDSolver**: Position Based Dynamics for cloth/soft bodies
   - **SFSolver**: Stable Fluids for smoke/gas
   - **AvatarSolver**: Special solver for animated characters
   - **ToolSolver**: Handles tool interactions

4. **Entities** (`genesis/engine/entities/`): Physical objects in the simulation
   - Base class `Entity` with specialized types for each solver
   - Support for loading various formats (URDF, MJCF, OBJ, etc.)
   - Material properties and physical parameters

5. **Materials** (`genesis/engine/materials/`): Material models for different solvers
   - Each solver has its own material types
   - Examples: elastic, plastic, liquid, muscle, cloth

6. **Visualization** (`genesis/vis/`): Rendering and interactive viewing
   - **Visualizer**: Main visualization controller
   - **Viewer**: Interactive PyOpenGL-based viewer
   - **Rasterizer/Raytracer**: Different rendering backends
   - Support for both headless and interactive modes

### Key Design Patterns

- **Taichi-based computation**: Uses Taichi for GPU-accelerated physics
- **Options pattern**: Extensive use of options classes for configuration
- **State management**: Separate state objects for checkpointing/rollback
- **Builder pattern**: Entities must be built after configuration
- **Global initialization**: `gs.init()` required before use

### Important Implementation Details

- Backend support: CPU, CUDA, Vulkan, Metal (experimental)
- Precision: Supports both float32 and float64
- Parallelization: Multi-environment simulation support
- Differentiability: MPM and Tool solvers support gradients
- File formats: URDF, MJCF, OBJ, GLB, STL, PLY, USD

### Critical Paths

1. **Simulation Loop**:
   ```
   Scene.step() -> Simulator.step() -> 
   [Solver.step() for each solver] -> Coupler.couple()
   ```

2. **Entity Creation**:
   ```
   Scene.add_entity() -> Entity.__init__() -> 
   Entity.build() -> Solver.add_entity()
   ```

3. **Rendering Pipeline**:
   ```
   Camera.render() -> Rasterizer/Raytracer -> 
   Mesh data -> Image output
   ```

### Performance Considerations

- Avoid CPU-GPU synchronization in tight loops
- Use batched operations where possible
- Profile with `profiling_options` for optimization
- Consider `performance_mode=True` for production

### Common Pitfalls

- Forgetting to call `entity.build()` after configuration
- Not calling `gs.init()` before creating scenes
- Mixing precision types (float32/float64)
- Creating entities after simulation has started