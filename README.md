# research-projects-RIT
Clean version of research-projects, just with ILE


## Running ILE

These are (poorly organized -- cleanup needed!) notes on running the ILE code. There's no garuntee they work, and steps are probably missing. They've been cobbled together from various sources. 

#### Example workflow

1. Create xml injection file (maybe no noise, mdc.xml.gz)

``` bash
>>> util_WriteInjectionFile.py --parameter m1 --parameter-value 100 --parameter m2 --parameter-value 30 --parameter incl --parameter-value 0.785398 --parameter dist --parameter-value 568.3 --parameter tref --parameter-value 1000000000 --approx EOBNRv2HM --parameter fmin --parameter-value 10
>>> util_PrintInjection.py --inj mdc.xml.gz --event 0 --verbose
```

2. Create frame file (frames.cache)... can probably source richard's directory, see "NR_plot"

```bash
>>> util_LALWriteFrame.py --instrument H1 --inj mdc.xml.gz --event 0 --start 999999900 --stop 1000001000
>>> util_LALWriteFrame.py --instrument L1 --inj mdc.xml.gz --event 0 --start 999999900 --stop 1000001000
>>> find . -name '*.gwf' | lalapps_path2cache > frames.cache
```

3. Check zero-noise SNR

```bash
>>> cp /home/scott.field/ILE-ROM/ReferenceRuns/synthetic_data/*psd* .
>>> /home/oshaughn/unixhome/PythonPackages/util_FrameZeroNoiseSNR.py --cache frames.cache --psd-file H1=H1-psd.xml.gz --psd-file L1=L1-psd.xml.gz
```

4. lays out grid in (M,q)...

```bash
>>> util_ManualOverlapGrid.py --inj mdc.xml.gz --event 0 --parameter 'mtot' --parameter-range '[110,160]' --parameter q --parameter-range '[0.1,1]' --skip-overlap --verbose --grid-cartesian-npts 500)
```

5. create a DAG: create_event_dag_via_grid

from run_standard...

```bash
>>> create_event_dag_via_grid --cache-file frames.cache   --event-time 1000000000  --sim-xml overlap-grid.xml.gz  --fmin-template 20 --reference-freq 0 --channel-name H1=FAKE-STRAIN --channel-name L1=FAKE-STRAIN  --psd-file "H1=H1-psd.xml.gz" --psd-file "L1=L1-psd.xml.gz"  --save-samples --time-marginalization --n-max 500000 --n-eff 200 --output-file CME.xml.gz    --n-copies 2 --fmax 2000 --adapt-weight-exponent 0.2 --adapt-floor-level 0.1 --n-chunk 4000 --approximant EOBNRv2HM --l-max 4 --declination-cosine-sampler --inclination-cosine-sampler  --save-P 0.9
```

6. submit the dag, and monitor its progress

```bash
>>> condor_submit_dag integrate.sub
>>> condor_q -dag
```

*** NOTE ***

```bash
>>> condor_submit integrate.sub  # works
>>> condor_submit_dag integrate.sub  # fails
>>> condor_submit_dag marginalize_extrinsic_parameters_grid.dag  # works
```
7. 

```bash
>>> cat run_standard_allmodes_v?_bajor.composite > tmp_allmodes.composite  # joined !
        mkdir run_standard_allmodes; exit 0
        (cd run_standard_allmodes; python ${HERE}/../../python/util_QuadraticMassPosterior.py --inj-file ../mdc.xml.gz --fname ../tmp_allmodes.composite   --coordinates-M-q --n-max 3e6 --n-eff 3000) #   --fit-method gp )
```

#### With makefile

These are the basic steps using makefiles. The steps described in Workflow (above) provide details on the key steps automated by the makefiles. If you're just starting out, please follow the workflow instead. 

```bash
make 
make frames.cache
make run_standard
make run_rom

cd into run_*
condor_submit_dag marginalize_extrinsic_parameters_grid.dag
condor_q
```
