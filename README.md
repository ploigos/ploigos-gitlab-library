# ploigos-gitlab-library

TODO, Section 1 (as pulled from top of workflow files):

## - separatePlatformConfig: need separate runners for true/false scenarios (including documentation on how/why)
    #    /* Directory into which platform configuration is mounted, if applicable */
    #    PLATFORM_CONFIG_DIR: "/opt/platform-config"
    #
    #    /* Additional mounts for agent containers, if separatePlatformConfig == true */
    #    String PLATFORM_MOUNTS = params.separatePlatformConfig ? """
    #          - mountPath: ${PLATFORM_CONFIG_DIR}/config.yml
    #            name: ploigos-platform-config
    #            subPath: config.yml
    #          - mountPath: ${PLATFORM_CONFIG_DIR}/config-secrets.yml
    #            name: ploigos-platform-config-secrets
    #            subPath: config-secrets.yml
    #    """ : ""
    #
    #    /* Additional volumes for the agent Pod, if separatePlatformConfig == true */
    #    String PLATFORM_VOLUMES = params.separatePlatformConfig ? """
    #        - name: ploigos-platform-config
    #          configMap:
    #            name: ploigos-platform-config
    #        - name: ploigos-platform-config-secrets
    #          secret:
    #            secretName: ploigos-platform-config-secrets
    #    """ : ""

    #    /* Combine this app's local config with platform-level config, if separatePlatformConfig == true */
    #    String PSR_CONFIG_ARG = params.separatePlatformConfig ?
    #        "${PLATFORM_CONFIG_DIR} ${params.stepRunnerConfigDir}" : "${params.stepRunnerConfigDir}"

## - trustedCABundleConfig should be known at platform level, not app level; the runner should know this, not the pipeline
    #    /* Additional mount for agent containers, if trustedCaConfig == true */
    #    String TLS_MOUNTS = params.trustedCABundleConfig ? """
    #          - name: trusted-ca
    #            mountPath: /etc/pki/ca-trust/source/anchors
    #            readOnly: true
    #    """ : ""

    #    /* Additional volume for agent containers, if trustedCaConfig == true */
    #    String TLS_VOLUMES = params.trustedCABundleConfig ? """
    #        - name: trusted-ca
    #          configMap:
    #            name: ${params.trustedCABundleConfigMapName}
    #            items:
    #            - key: ca-bundle.crt
    #              path: tls-ca-bundle.pem
    #    """ : ""

## Other pod configs that look like they belong with runner?? Might be able to configure in pipeline...
##   imagePullPolicy: "${params.workflowWorkersImagePullPolicy}"
##   tty: true
##   imagePullPolicy: "${params.workflowWorkersImagePullPolicy}"

## - `command: ['sh', '-c', 'update-ca-trust && cat']`; how do we make this happen in GitLab

## - Jenkins / Tekton workflows have pod labels based on variables, but GitLab doesn't carry the same functionality:
##     - https://docs.gitlab.com/runner/install/kubernetes.html#set-pod-labels-to-ci-environment-variables-keys

##
##
## NOTE: Branch-matching regex expressions are hard-coded and duplicated at the moment, due to an
##       open issue with GitLab: https://gitlab.com/gitlab-org/gitlab/-/issues/35438
##
## NOTE: Rules pulled in from the extends cannot be merged, so must duplicate here; see:
##       https://docs.gitlab.com/ee/ci/yaml/#merge-details
##       https://github.com/yaml/yaml/issues/48

TODO, Section 2:

  - DOCUMENTATION: This README needs to be super-awesome like the other two runners

  - DOCUMENTATION: Explicitly spell out what the minimal / standard pipeline look like, what the imported workflow looks like, and how the files in the repo with the workflow are laid out.

  - Separate out the `config/` dir to a separate repo, but be sure the necessary pieces are documented in this README

  - NOTE: See TODO section in gitlab-ci-minimal.yml (need to move TODOs out of there later anyway)

  - Need to manually add all dirs under '/builds' that need to pass from step to step, until GitLab Runner 13.12+ is installed (see the note on this below)

  - The "setup_workflow_step_runner" job mounts '/home/ploigos', but it's not dynamic, so pulls the previous build. Need to clean the folder, but the find+rm takes forever. Can't use rm -rf due to pipefail when trying to remove .* (attempts to remove . and .., then fails script). NOTE: Can we just rm -rf specific folders we know will be huge, *then* call the find+rm??

  - DOCUMENTATION: List out hard-coded values that cannot be set as variables, and where they live (so far: URLs for include; regex for rules)

  - DOCUMENTATION: When setting up the GitLab CI Runner in OpenShift, load 'config.toml' into a ConfigMap (see config/config-toml.yml).

NOTE: Until GitLab CI Runner 13.12+ can be loaded on OpenShift, artifacts will be passed between steps. As of 13.12, the `/builds` folder can be mounted in a PVC, and passed between jobs.


Considerations for setting up a GitLab CI Runner for Ploigos:

* The runner tags MUST be hard-coded, and cannot use a variable value (see: https://gitlab.com/gitlab-org/gitlab-foss/-/issues/24207 ). To work around this, one job can be created for each combination of tags. Rules can then be set based on the desired variables to pick up the correct job, which will have the appropriate tags hard-coded.

* Unlike Jenkins/Tekton pipelines, the GitLab CI Ploigos implementation does not allow for the CA bundle to be dynamically chosen per pipeline; this should be decided upon as a platform-level config in advance, and baked into the Runner as appropriate.


NOTE: Human-readable job names can't be set; GitLab CI will always parse the job name, and possibly make minor changes to it (e.g., forced capitalization of the first letter of the job). See: https://gitlab.com/gitlab-org/gitlab/-/issues/23672