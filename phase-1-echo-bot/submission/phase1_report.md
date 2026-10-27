# Phase 1 — Comparison Report

Copy this file to `submission/phase1_report.md`, fill in every answer slot, and commit it:

```bash
cp phase-1-echo-bot/report_template.md submission/phase1_report.md
```

**How to fill it in.** Each answer sits between a pair of `<!--answer:...-->` markers. Replace
the `TODO` line with your answer — **leave the markers themselves untouched**, they are how the
grader finds your answers. Anything you write outside the markers is ignored, so add extra
prose, tables or images freely.

Numbers must **match `submission/phase1_report.json`** exactly — that file is produced by
`measure.py` and is what your figures are checked against. Copy them across; do not retype from
memory.

---

## 1. Setup facts

**Your image size**, in MB, from `gcloud artifacts docker images list` (number only, e.g. `142`):

<!--answer:image_size_mb-->
50
<!--/answer-->

**The custom VPC and subnet you created** (names only):

<!--answer:vpc_name-->
echo-net
<!--/answer-->

<!--answer:subnet_name-->
echo-subnet
<!--/answer-->

---

## 2. Measurements

Copy these from `submission/phase1_report.json`. Cold is `cold_ms`; warm median is
`stats.median`. Numbers only, in milliseconds (e.g. `812.4`).

| Platform | Cold (ms) | Warm median (ms) |
|---|---|---|
| IaaS VM | <!--answer:vm_cold_ms-->269.6<!--/answer--> | <!--answer:vm_warm_median_ms-->221.7<!--/answer--> |
| Cloud Run | <!--answer:cloudrun_cold_ms-->657.6<!--/answer--> | <!--answer:cloudrun_warm_median_ms-->383.4<!--/answer--> |
| Cloud Function | <!--answer:function_cold_ms-->137.3<!--/answer--> | <!--answer:function_warm_median_ms-->131.5<!--/answer--> |

Include your histogram or plot of the warm distribution below (an image, or the ASCII chart
`measure.py` prints). This is read by a human, not the grader:

<!--answer:latency_plot-->
Phase 1 — deployment measurements
------------------------------------------------------------
[OK  ] vm        cold=  269.6ms  warm_median=  221.7ms
            #######################
[OK  ] cloudrun  cold=  657.6ms  warm_median=  383.4ms
            ########################################
[OK  ] function  cold=  137.3ms  warm_median=  131.5ms
            ##############
<!--/answer-->

---

## 3. Per-platform analysis

**Scaling behaviour.** Imagine requests arriving one at a time, then ten at once, then a
hundred. For each of the three platforms, answer concretely:

- Does anything get created automatically to absorb the load — and if so, *what* exactly?
  (One platform here creates nothing at all.)
- What is the hard limit, and where does it come from — a flag you passed yourself, or the
  size of the machine you picked?

(~80 words. Your `--machine-type=e2-micro` and `--max-instances=3` are both concrete answers
to the second question.)

<!--answer:scaling_behaviour-->
For the IaaS VM, nothing is created automatically to absorb load; requests simply queue, and the hard limit is dictated by the machine's fixed hardware size (--machine-type=e2-micro), leading to resource saturation. Conversely, Cloud Run and Cloud Functions automatically provision new container instances to absorb concurrent traffic spikes. Their hard limit comes from the explicit configuration flag you passed (--max-instances=3), which restricts maximum instance scaling.
<!--/answer-->

**Deployment effort.** What did you have to build, configure and maintain to get each of the
three running? (~100 words)

<!--answer:deployment_effort-->
Deploying on the IaaS VM required high manual effort: setting up a custom VPC and subnet, configuring network access, installing Docker, pulling the container image, and running it manually. In contrast, Cloud Run required only pushing the image to Artifact Registry and executing a single deployment command, with zero infrastructure maintenance. Cloud Functions utilized a similar serverless workflow. While Cloud Run and Functions entirely offload server patching, scaling, and provisioning to Google Cloud, the VM demands continuous manual monitoring and management.
<!--/answer-->

---

## 4. Why do two of your three numbers look alike?

With Cloud Run and your Function both deployed, run `gcloud run services list` and paste the
**complete output** here, unedited:

<!--answer:run_services_list-->
✔
SERVICE: echo-bot
REGION: us-central1
URL: https://echo-bot-808604068446.us-central1.run.app
LAST DEPLOYED BY: tess.etienne-moreau@eurecom.fr
LAST DEPLOYED AT: 2026-10-27T15:57:08.620305Z

✔
SERVICE: echo-function
REGION: us-central1
URL: https://echo-function-808604068446.us-central1.run.app
LAST DEPLOYED BY: service-808604068446@gcf-admin-robot.iam.gserviceaccount.com
LAST DEPLOYED AT: 2026-10-27T16:06:01.568859Z
<!--/answer-->

Now explain what that output tells you about how your Function is actually being run, why its
cold start resembles Cloud Run's so closely, and what — if anything — genuinely differs between
the two deployment paths. (~120 words)

<!--answer:convergence_explanation-->
The output shows that both `echo-bot` (Cloud Run) and `echo-function` (Cloud Functions) appear side by side as services managed by Cloud Run. This confirms that Cloud Functions Gen 2 is built directly on top of Cloud Run infrastructure under the hood. Consequently, your Cloud Function runs as a containerized service inside Cloud Run's environment. This architectural convergence explains why their performance metrics and cold starts resemble each other so closely—both rely on the same underlying serverless container execution model. The primary difference lies in the deployment path: Cloud Run gives you direct control over the raw container service configuration, whereas Cloud Functions abstracts packaging through a function-centric framework managed by Google Cloud service accounts.
<!--/answer-->
