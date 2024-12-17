// The logic for every repo:
// 1. clone and throw the clone away unless a preserved copy is needed
// 2. pull to the preserved copy
// 3. refresh 2, 3, 5
//

// global variables
//   docker_image
//   docker_params

def saved_dir = "${env.SAVED_DIR}"

timestamps {
  stage('Prepare') {
    node ('debbies') {
      writeFile file:'Dockerfile', text: '''
FROM python:3.10

RUN pip3 install --upgrade pip
RUN pip3 install pyFreenet3
RUN pip3 install 'mercurial<6'
RUN git clone http://localhost:8888/freenet:$(curl http://localhost:8888/freenet:USK@Mm9MIkkeQhs~OMiCQ~83Vs48EvNwVRxjfeoFMOQHUYI,AxOZEuOyRM7oJjU43HFErhVw06ZIJLb8GMKNheWR3g4,AQACAAE/infocalypse/0/ | sed '/Permanent/s/.*freenet://;s/".*//') /usr/local/src/infocalypse
RUN echo "[extensions]" >> $HOME/.hgrc
RUN echo "infocalypse=$(pwd)/infocalypse/infocalypse" >> $HOME/.hgrc
RUN hg fn-setup --nofms --nowot
  ''';
      docker_image = docker.build('hgfreenet:3', '--network=host .');
      docker_params = "--network=host";
    }
  }
}

def wait_count = 0

def gen_cl = { project, key ->
  boolean perm_done = false
  boolean toss_done = false
  boolean reinsert_done = false
  int reinsert_level = 2
  int wait_laps_1_left = ++wait_count
  int wait_laps_2_left = wait_count
  int wait_laps_3_left = wait_count
  return {
    def perm_dir = "$saved_dir/perm-$project"

    if (!perm_done) {
      if (wait_laps_1_left-- > 0) {
          return wait_count
      }

      def dir = perm_dir
      sh "test -d ${dir} || hg clone freenet:${key} ${dir}"
      sh "cd ${dir} && hg pull"
      sh "cd ${dir} && hg log | egrep . > /dev/null"
      perm_done = true
      return 2000
    }

    if (!toss_done) {
      if (wait_laps_2_left-- > 0) {
          return wait_count
      }

      def dir = "/tmp/throwaway-$project"
      sh script: "test -d ${dir} && rm -r ${dir}", returnStatus: true
      sh "hg clone freenet:${key} ${dir}"
      sh "rm -r ${dir}"
      toss_done = true
      return 2000
    }

    if (!reinsert_done) {
      if (wait_laps_3_left-- > 0) {
          return wait_count
      }

      if (reinsert_level == 4) {
        // Skip level 4 of reinsert
        reinsert_level++
      }
      def level = reinsert_level++
      def dir = perm_dir
      sh "cd ${dir} && hg fn-reinsert --level $level"
      if (level < 5) {
          wait_laps_3_left = wait_count++
          return 2000
      }
      reinsert_done = true
    }

    return 0
  }
}

def buildParallelMap = [:]
def to_process = [
		  "infocalypse":                   "USK@6~ZDYdvAgMoUfG6M5Kwi7SQqyS-gTcyFeaNN1Pf3FvY,OSOT4OEeg4xyYnwcGECZUX6~lnmYrZsz05Km7G7bvOQ,AQACAAE/infocalypse.R1/31",
		  "fred":                          "USK@yJUguKTfUHgVutplApc8A3ucq~QogPfqx3-1ZunKjYk,EzZzXErTnhC~ll7HGpgDDik15KTFlwdpuGcRA7HL5uk,AQACAAE/fred.R1/0",
		  "pyFreenet":                     "USK@-sEkUkv7SClGNhjpvFF9TtmGHnoiwvThkepMUTy7Fh4,Rciq6x0804hb~rNzxmINfkuSCgAEvHZjJNHQcUo0nuQ,AQACAAE/pyfreenet.R1/0",
		  // "pyFreenet":                     "USK@dqWzp0iGflRepXyBXHzxyKMSxq90kP2Lof8EdEr6woQ,nL53lCqG5ssdNtXMtVzTePJ4QYWGLkAhWMdmRqxwFjw,AQACAAE/pyFreenet.R1/1",
		  "hg-git":                        "USK@hRSO9yNEENHalLqm8KCjn0XCSk-DlLnALp4CaXunjLs,TNRUMEkbKtIPNJPf-HV9ykWl4cEcmIob3wFluZZzYGk,AQACAAE/hg-git.R1/8",
		  // "hg-git":                        "USK@BTOaKIcNsNoa-z0qIAjXI0WAN25tNru8GtMCSsZh-sk,cZUVEEzoud2cdFHtrf4EC-EKsjvHBMRwvAzebqe4fjM,AQACAAE/hg-git.R1/1",
		  "b":                             "USK@6~ZDYdvAgMoUfG6M5Kwi7SQqyS-gTcyFeaNN1Pf3FvY,OSOT4OEeg4xyYnwcGECZUX6~lnmYrZsz05Km7G7bvOQ,AQACAAE/b.R1/1",
		  "infocalypse-dvcs-over-freenet": "USK@ugGhE7lfI-QyFYL6yuZt7uyhlrVfvIRo0xkQxd2qV~c,L1bCwDm5Hh5qgqml7SBQcF85vL8IPvzN4WihjWIKH8I,AQACAAE/infocalypse-dvcs-over-freenet.R1/15",
		  "wiki_hacking":                  "USK@kRM~jJVREwnN2qnA8R0Vt8HmpfRzBZ0j4rHC2cQ-0hw,2xcoQVdQLyqfTpF2DpkdUIbHFCeL4W~2X1phUYymnhM,AQACAAE/wiki_hacking.R1/24",
		  "infocalypse-repo":              "USK@j2Mfv0b81MNSslh3Qg9gs1vgQj36whsuosP8ydZq158,Xb~vh8XEXmdwGFca4HAx-ILOZD6AWPVuRzIuLerl4PU,AQACAAE/infocalypse-repo.R1/20",
		  "fred1":                         "USK@3Xvf~RI-N8YC-xFtQIEnEwR~ZLC05UhQjQzKAS0lL4Y,PqTg9Y5Me6~WthiW5FoHG9Mq18c7oRbe0owRutJJJZc,AQACAAE/fred1.R1/124",
		  ]
timestamps {
  to_process.each { project, key ->
    buildParallelMap[project] = {
      stage(project) {
	def cl = gen_cl(project, key)
	def result = 1
	// The closure cl is run using cl() on the node
	// When it cannot do anymore (completed or needs to
	// sleep or wait for something) it returns.
	// If a positive value is returned, it sleeps this many seconds.
	// This allows others to use the node while this closure doesn't.
	while ({
	  node ('debbies') {
	    docker_image.inside(docker_params) {
	      result = cl()
		}
	  }
	  result > 0
	    }()) {
	  sleep(result)
	}
      }
    }
  }

  parallel(buildParallelMap)
}
