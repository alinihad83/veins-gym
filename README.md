Veins-Gym
=========

Veins-Gym exports Veins simulations as Open AI Gyms.
This enables the application of Reinforcement Learning algorithms to solve problems in the VANET domain, in particular popular frameworks such as Tensorflow or PyTorch.

To install, simply run `pip install veins-gym` ([Veins-Gym on PyPI](https://pypi.org/project/veins-gym/)).

License
-------
This project is licensed under the terms of the GNU General Public License 2.0.


Hints and FAQ
-------------

Here are some frequent issues with Veins-Gym and its environments.

Please also checkout the guide on how to build new environments in [doc/getting_started.md](https://github.com/tkn-tub/veins-gym/blob/master/doc/getting_started.md).


### Protobuf Files Out of Sync

Sometimes the C++ code files generated by `protoc` get out of sync with the installed version of `libprotobuf`.
This leads to errors at link-time of the environment (not veins-gym itself, which does not contain C++-code) like:
```
undefined reference to google::protobuf::internal::ParseContext::ParseMessage
```

**Solution:**
Re-generate the `.pb.cc` and `.pb.h` files in the environment.

* check/update the installation of `protobuf-compiler` and `libprotoc` on your system (package names may differ based on the Linux distribution)
* remove the generated `src/protobuf/veinsgym.pb.h` and `src/protobuf/veinsgym.pb.cc` files (or however they may be called in the environment)
* re-generate the protobuf-files by running `snakemake src/protobuf/veinsgym.pb.cc src/protobuf/veinsgym.pb.h` (adapt for the files in your environment.

Also see [Issue #1 of serpentine-env](https://github.com/tkn-tub/serpentine-env/issues/1).


### Showing the (Debug) Output of Veins through the Agent

Normally, Veins-Gym swallows the standard output of veins simulations started by its environments.
This reduces output clutter but makes debugging harder as error messages are not visible.

**Solution:**
Enable veins standard output by adjusting `gym.register` when running your agent:

```python
gym.register(
	# ...
	kwargs={
		# ...
		"print_veins_stdout": True,  # enable (debug) output of veins
	}
)
```

### Starting Veins Simulations Separately

Sometimes you want to run Veins simulations separately, e.g., with custom parameters, in a (different) container, or within a debugger.
This is hard to achieve and less flexible when Veins-Gym launches the Veins simulation, as it does by default.

**Solution:**
Disable auto-start of Veins in your agent's environment:

```python
gym.register(
	# ...
	kwargs={
		# ...
		"run_veins": False,  # do not start veins through Veins-Gym
		"port": 5555,  # pick a port to use
	}
)
```

Then run your veins simulation manually, typically from a `scenario` directory, in which the `omnetpp.ini` file is located:
```bash
./run -u Cmdenv '--*.gym-connection.port=5555'
```

Make sure to use the same port there.
Also, consider starting Veins first in order to avoid timeouts.


### Avoid Timeouts

Veins-Gym expects a request from the Veins simulation after a certain time after `env.reset` (which may in turn have started the Veins simulation).
If the Veins simulation takes a longer amount of (wall-clock) time to reach the point where the request is sent, the agent's environment may have timed out.
This is intended to notify about stuck simulations, but may not work as intended if the simulation is more complex or the host' hardware is less powerful than expected.

**Solution:**
Increase the timeout when registering the gym environment.

```python
gym.register(
	# ...
	kwargs={
		# ...
		"timeout": 10.0,  # new timeout value (in seconds)
	}
)
```
