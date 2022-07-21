Agenda for July 26 (Tuesday):
- PyCharm
  - Presentation (@liamhuber)
    - Homework check: verify that PyCharm is installed and has access to a pyiron-capable interpreter
    - Download a demo notebook
    - _New Project_
      - Create a new project from scratch
        - Choose our pyiron conda env (or create a new env if they failed homework)
        - Open "Python Packages" window and verify that the env uses `https://conda.anaconda.org/conda-forge/` as its only repository, and that the latest versions of `pyiron` and `jupyter` installed (or set and install if they failed homework)
      - Move the notebook into project
      - Launch jupyter from PyCharm terminal (or within PyCharm if you have pro version)
      - Walk through the notebook
    - _From Notebook To Module_
      - Create a python module inside the project
      - Iteratively move code from the notebook into the module, and access it via imports in the notebook (Some more detail here once I've made the notebook, but sub-steps will make sure we practice relevant material in the learning objectives)
    - _Development with Git_
      - Clone `pyiron_continuum` onto local machine
      - Check out `tutorial` branch
      - Make a new branch off that
      - Add an empty file to hold the workflow you will make later in the workshop
      - Commit and push
  - Learning objectives:
    - Manage your python (conda) environment from inside PyCharm
    - Recognize and use Editor features: syntax highlighting, error highlighting, autocompletion
    - Navigate PyCharm using the Project and Structure tabs and by control-clicking
    - Use Git from directly within PyCharm: create, pull, commit, push, ~merge~ (do later), fetch
    - Know the difference between searching content by usage and by string matching and be able to do both
    - Know where to find the refactoring tab, and use it for simple tasks like renaming variables
- `pyiron_base` (@jan-janssen)
  - `Project`
  - `job.status`
  - `job.run` (how different runs are related to `job.status`)
  - `job.server`
  - `job.executable`
  - `GenericMaster`
- `pyiron_atomistics` (@samwaseda)
  - LAMMPS
  - VASP
  - SPHInX
  - Murnaghan
- Writing own workflows (all) ~1h
- Presentation of workflows on GitHub

Agenda for July 27 (Wednesday)
- GitHub (Comments on PR, issues, review etc.) (@srmnitc)
- Getting a dev environment setup on the cluster (@prmv)
  - local check out of repositories
  - remote and local debugging and profiling
- Refactoring (@niklassiemer)
  - pycharm tools
  - pycharm highlighting
  - pep8
  - zen of python and where to find it
- Unit tests & DocStrings (@pmrv)
- Correct + peer-review of PRs (from Day I)
- Introduction to easy issues (cf. [issue page](https://github.com/pyiron/pyiron_atomistics/issues))
- Discussion
- Invitation to pyiron meetings

Homework: before coming they should
- go on google and know the following terms in the context of python: "function", "argument", "class", "method", "property", "module". Bonus term: "decorator"
- install pycharm
- install pyiron (`conda install pyiron`)

Feel free to modify this post if necessary.
