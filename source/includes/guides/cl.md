<h1 class="title topictitle1">Adding a node to a cluster</h1>
<div class="body refbody"><p class="shortdesc">Use these instructions to add a node to a cluster and rebalance
the shards.</p>
<div class="section"><h2 class="title sectiontitle">Step 1: Provision the new node</h2>     	<p class="p">Install
Cloudant Local on the database node that you are adding, as described
in <a class="xref" href="clinstall_extract_install_cloudant_local.html" title="Use these instructions to extract and install Cloudant Local on a database or load balancer node in either a multi-node cluster or a single-node implementation. If needed, you also can use these instructions to install Cloudant Local on a mount point or move the product to a mount point if it is already installed.">Extracting and Installing Cloudant Local</a>.</p>

			<div class="p">Configure the node as described in <a class="xref" href="clinstall_configure_cloudant_local.html#clinstall_configure_cloudant_local" title="Use these instructions to configure Cloudant Local on a database or load balancer node in either a multi-node cluster or a single-node implementation. You must configure Cloudant Local on each database and load balancer node in your implementation.">Configuring Cloudant Local</a>, but with the following modifications to the configuration process:<ol class="ol"><li class="li">Update the <span class="ph filepath">configure.ini</span> file. Confirm that the
							<span class="ph filepath">configure.ini</span> file on this node has correct and
						complete information about the cluster. Check the names and IP addresses of
						the database and load balancer nodes, username and passwords, encrypted
						passwords and the cookie value. If required, refer to or copy the last
						updated version of the <span class="ph filepath">configure.ini</span> file that might
						exist on the other database nodes in the directory
							<span class="ph filepath">/root/Cloudant/repo</span>. Ensure that the file is
						accurate and complete.</li>
<li class="li">Add the name and IP address of this new node to the database node section in
						the <span class="ph filepath">configure.ini</span> file.</li>
<li class="li">Run the <span class="ph filepath">configure.sh</span> script with a <var class="keyword varname">–m</var> argument to
						configure and start the node in maintenance
						mode:<pre class="pre codeblock"><code>configure.sh -m</code></pre>
The node to be added must be
						started in maintenance mode. The <var class="keyword varname">–m</var> argument in the
						command configures the node based on the <span class="ph filepath">configure.ini</span>
						file, and starts the node in maintenance mode.</li>
</ol>
</div>
</div>
<div class="section"><h2 class="title sectiontitle">Step 2: Confirm that the node is in maintenance mode</h2> 		  			<div class="p">The node must be in maintenance mode so that it does not cause any errors when it is brought
				into the cluster. Confirm that the node is in maintenance mode. Use the following
				command to confirm that the node is in maintenance
				mode:<pre class="pre codeblock"><code>curl localhost:5984/_up</code></pre>
If the node is in maintenance
				mode, you receive the following
				response:<pre class="pre codeblock"><code>{
    "status": "maintenance_mode"
}</code></pre>
You can also
				manually place the node in maintenance mode by adding the
				line:<pre class="pre codeblock"><code>maintenance_mode = true</code></pre>
in the
					<span class="keyword cmdname">[couchdb]</span> section of the
					<span class="ph filepath">/opt/cloudant/etc/local.ini</span> file, then restarting
				Cloudant.</div>
</div>
<div class="section"><h2 class="title sectiontitle">Step 3: Add the new node to the nodes database on one of the existing
				nodes</h2>			
			<p class="p">SSH into one of the existing database nodes and add the new node to its
					<samp class="ph codeph">nodes</samp> database:</p>

			<p class="p"><span class="keyword cmdname">curl 'localhost:5986/nodes/cloudant@db4.cluster001.cloudant.net' -X PUT -d
					'{}'</span></p>

			<p class="p">Use this command to verify that the node was successfully added to the
					<samp class="ph codeph">nodes</samp> database:</p>

			<p class="p"><span class="keyword cmdname">curl -u admin:password
					http://localhost:5986/nodes/_all_docs?include_docs=true</span></p>

			<p class="p">If the node was added to the <samp class="ph codeph">nodes</samp> database, the output from this command
				includes the new node in the resulting nodes list. The list also includes the
				document ID specified for each database node in the cluster.</p>

			<p class="p">Use this command to verify that membership for the new node was replicated to every
				database node in your cluster:</p>

			<p class="p"><span class="keyword cmdname">curl -u </span><var class="keyword varname">admin:password</var><span class="keyword cmdname">					http://</span><var class="keyword varname">hostname</var><span class="keyword cmdname">:5984/_membership</span></p>

			<p class="p">Replace <var class="keyword varname">admin:password</var> in the preceding command with the
				credentials for the database administrator. Also, replace
					<var class="keyword varname">hostname</var> with the fully qualified domain name for the
				database node.</p>

			<p class="p">Run the preceding curl command for every node to confirm that membership for the new
				node is included in the output for the other nodes in your cluster.</p>

		</div>
<div class="section"><h2 class="title sectiontitle">Step 4: Update the load balancer configuration</h2> 			<p class="p">The load balancers must know to direct traffic to the new node. For each load balancer, add the
				new node to the list of nodes in the <span class="ph filepath">/etc/haproxy/haproxy.cfg</span>
				file. You must restart haproxy after the changes.</p>
<p class="p">Verify
that your load balancer can correctly access the new node.</p>
<div class="note note"><span class="notetitle">Note:</span> Configuration
changes and restarts to the load balancer can affect traffic. How
these changes are made depends on your setup. If you are running a
failover load balancer, confirm that it is running correctly and you
can access the cluster through it. Then, make any required changes,
such as DNS settings, to reroute all traffic through the failover
load balancer.</div>
</div>
<div class="section" id="clinstall_add_node__plan"><h2 class="title sectiontitle">Step 5: Create a rebalancing plan</h2><p class="p">The next step is to rebalance
				the cluster. </p>
<p class="p">If you are using more than one load balancer, you can run the
				rebalancing process from any one of your load balancers. If you do not have a
				configuration file for the rebalancer, create a file named
					<span class="ph filepath">rebal.ini</span> and save it in the home directory for the user on
				the load balancer. The format of the configuration file is as
				follows:</p>
<pre class="pre codeblock"><code> 
[cloudant]
\# System user which runs the db process
db_user = cloudant
\# Server-wide database admin credentials. These have to be identical for every node of the cluster.
rebal_user =
rebal_password =
\# Default filename for rebalance plans.
rebal_plan = rebalance_plan
\# Mapping between public and private hostnames for nodes. This is only needed if
\# the host names have a private and a public interface and the load balancer does
\# not have access to the private interface. If the private host name can be used,
\# these fields can be left blank. Otherwise, the private host name extension
\# is replaced with the public host name extension to arrive at the public host
\# name. If you are unsure, try leaving these empty.

private_host_ext =
public_host_ext =
[rebal]
\# Set to true to disable prompts on failure and just continue
batch_mode = false
 </code></pre>
<p class="p">Confirm
				that the admin credentials for <var class="keyword varname">rebal_user</var> and
					<var class="keyword varname">rebal_password</var> in the configuration file are correct by
				logging in to one of the database nodes with SSH, and then run this curl
				command:</p>
<p class="p"><span class="keyword cmdname">curl
					http://</span><var class="keyword varname">admin:password</var><span class="keyword cmdname">@localhost:5986/_active_tasks</span></p>
<p class="p">Replace
					<var class="keyword varname">admin:password</var> in the preceding command with the credentials
				for the database administrator. If you specify the wrong credentials, you receive an
				error message.</p>
<div class="sectiondiv"><strong class="ph b">Setting up remote access for the rebalancer</strong>
				<p class="p">The rebalancer runs on the load balancer and needs SSH access to the database
					nodes to execute the operations that are required to move shards between
					database nodes. The Cloudant Local installer creates a
						<samp class="ph codeph">cloudantrebal</samp> user account on every database node in your
					cluster for the rebalancer. During installation, this account is configured with
					the permissions and path that are required to do the operations that the
					rebalancer needs to do with SSH. The <samp class="ph codeph">cloudantrebal</samp> account
					eliminates some of the work an operator must do to set up a cluster
					rebalance.</p>
<p class="p">Follow these instructions to enable SSH access for the
					rebalancer:</p>
<ol class="ol"><li class="li p" id="clinstall_add_node__enablesshaccessstep1">On the load balancer, generate a
						pair of public and private keys for SSH access to the rebalancer. This step
						can be done as the root user. For
						example:<pre class="pre codeblock"><code>&gt; cd ~/.ssh
&gt; ssh-keygen -t rsa</code></pre>
Enter a name
						for the key file, or leave it as the default. Record the passphrase that you
						enter for the key creation. The result is to produce two key files in the
							<span class="ph filepath">~/.ssh</span> folder:<ul class="ul"><li class="li">A private key file with the default name
								<span class="ph filepath">id_rsa</span>.</li>
<li class="li">A public key file with the default name
									<span class="ph filepath">id_rsa.pub</span>.</li>
</ul>
</li>
<li class="li p">On every database node in your cluster, create an
							<span class="ph filepath">authorized_keys</span> file in
							<span class="ph filepath">/opt/cloudantrebal/.ssh</span>. Then, add the public key
						generated in step <a class="xref" href="#clinstall_add_node__enablesshaccessstep1">1</a> to that file.</li>
<li class="li p">On the load balancer, create the following entry in the
							<span class="ph filepath">~/.ssh/config</span> file for the account that you are
						using to run the
						rebalancer:<pre class="pre codeblock"><code>Host *.yourcluster.yourdomain.com
    User cloudantrebal</code></pre>
</li>
<li class="li p">If you use <samp class="ph codeph">ssh-agent</samp> to manage the SSH
						credentials for the rebalancer, ensure that the private key used in step
							<a class="xref" href="#clinstall_add_node__enablesshaccessstep1">1</a> is
						specified by providing the file name of the private key to
							<samp class="ph codeph">ssh-add</samp>. For
						example:<pre class="pre codeblock"><code>&gt; eval $(ssh-agent)
&gt; ssh-add ~/.ssh/id_rsa</code></pre>
Enter
						the passphrase of the key when prompted. These steps save the passphrase so
						that you are not prompted for it again in the current <span class="keyword cmdname">ssh</span>
						session. You would run the commands again if you open a new
							<span class="keyword cmdname">ssh</span> session on the load balancer, for example when
						you run the rebalance shards scripts in subsequent steps.</li>
<li class="li">Confirm that the database nodes are accessible with SSH from the load
						balancer, such as by configuring <samp class="ph codeph">ssh-agent</samp>. You can verify
						the access by trying to SSH into all database nodes from the load
						balancer.</li>
</ol>
<div class="p">After the rebalancer is configured and the <samp class="ph codeph">ssh-agent</samp> is set
					up, you can start the rebalancing process by creating a rebalancing plan: <ol class="ol"><li class="li">Create a working directory to hold the plan and log files and change to
							that directory by specifying these commands:
							<pre class="pre codeblock"><code>mkdir rebaldir
cd rebaldir</code></pre>
</li>
<li class="li">Next, create the plan by running <samp class="ph codeph">rebal plan expand</samp> with
							one of the database nodes as an argument, as shown in the following
							example:<pre class="pre codeblock"><code>rebal plan expand 'db1.cluster001.example.com'</code></pre>
</li>
<li class="li">Check the output from this command for any error messages similar to
							this one: <pre class="pre codeblock"><code>Retrying SSH connection</code></pre>
 If you receive
							a similar error message, check the <samp class="ph codeph">ssh-agent</samp>
							configurations and confirm that you can SSH to all database nodes,
							without specifying a password.</li>
<li class="li">Your current directory now contains a
								<span class="ph filepath">rebalance_plan</span> file that includes a list of
							shard moves.</li>
</ol>
</div>
</div></div>
<div class="section"><h2 class="title sectiontitle">Step 6: Put the new node into production mode</h2><p class="p">You must take the new node
				out of maintenance mode so it can serve traffic for the moved shards, while the
				rebalancing is in progress.</p>
<p class="p">Use the following command in
					<samp class="ph codeph">remsh</samp> to put the node in production mode:</p>
<div class="p">				<pre class="pre codeblock"><code>config:set("couchdb", "maintenance_mode", "false").</code></pre>

				<div class="note note"><span class="notetitle">Note:</span> Pay close attention to the '<samp class="ph codeph">.</samp>' character at the end of the
					command.</div>

			</div>
<div class="note important"><span class="importanttitle">Important:</span> <samp class="ph codeph">remsh</samp> is a UNIX and Linux command that is
				used to execute a specified command at a remote host or to log in to the remote
				host. Before you use <samp class="ph codeph">remsh</samp>, you must configure it for Cloudant
				Local usage, as described in <a class="xref" href="clinstall_configure_remsh.html" title="Use these instructions to configure remsh for Cloudant Local. Remsh is a UNIX and Linux command that is used to run a specified command at a remote host or to log in to the remote host. Remsh is used to do various maintenance tasks.">Configuring remsh for Cloudant Local</a>.</div>
Run the following command on the new database node to verify the status of
			that node:<div class="p">				<pre class="pre codeblock"><code>curl localhost:5984/_up</code></pre>

			</div>
<div class="p">If the node is functioning normally in production mode, you receive the
				response:<pre class="pre codeblock"><code>{"status":"ok"}</code></pre>
</div>
</div>
<div class="section" id="clinstall_add_node__run_rebal"><h2 class="title sectiontitle">Step 7: Run rebalancing</h2><p class="p">Before you continue, confirm that
				there are no other shard moves being executed on the same cluster. Moving multiple
				shards of the same database concurrently can lead to data loss.</p>
<p class="p">Use the
				following command to run your rebalancing plan:</p>
<div class="p">				<pre class="pre codeblock"><code>rebal run <var class="keyword varname">rebalance_plan_filename</var></code></pre>

			</div>
<p class="p">Replace <var class="keyword varname">rebalance_plan_filename</var> with the file name for your
				rebalance plan. The file name was specified earlier when you edited the
					<span class="ph filepath">rebal.ini</span> file while <a class="xref" href="#clinstall_add_node__plan">creating a rebalancing plan</a>.</p>
<div class="p">The command executes the
				shard moves identified in the plan. Shards moves are batched by database name with
				each batch being executed in series. Multiple batches are executed concurrently,
				with batches of 20 as the default. Progress and errors are logged to the
					<span class="ph filepath">rebalance_plan.log</span>. You can follow the progress by using
				the <span class="keyword cmdname">tail</span> command:
				<pre class="pre codeblock"><code>tail -f rebalance_plan.log</code></pre>
</div>
<p class="p">As the command progresses, 2
				log files are created for each database, <span class="ph filepath">dbname.out</span> and
					<span class="ph filepath">dbname.err</span>. The files store standard output and error
				messages, respectively.</p>
<div class="sectiondiv"><strong class="ph b">What to do if an error occurs.</strong>
				<div class="p">If an error occurs, do the following steps:<ul class="ul"><li class="li">Check the log files for the presence of <span class="keyword cmdname">sudo</span> or
								<span class="keyword cmdname">tty</span> errors. If they are present, do the following
							tasks for the <span class="ph filepath">/etc/sudoers</span> file on all the database nodes:<ol class="ol"><li class="li">Search for a line that
									contains:<pre class="pre codeblock"><code>Defaults requiretty</code></pre>
Modify the
									line to read:<pre class="pre codeblock"><code>Defaults !requiretty</code></pre>
</li>
<li class="li">Add the following lines to the file:
									<pre class="pre codeblock"><code>cloudantrebal        ALL=(ALL) NOPASSWD: ALL
%cloudant            ALL=(ALL) NOPASSWD: ALL</code></pre>
</li>
</ol>
</li>
<li class="li p">It might be a problem with the configuration of
								<samp class="ph codeph">rebal</samp> or <samp class="ph codeph">ssh-agent</samp>. Confirm that
								<samp class="ph codeph">ssh-agent</samp> is set up correctly and that the
								<span class="ph filepath">.rebal</span> file contains the right
							credentials.</li>
<li class="li p">If you see many <samp class="ph codeph">erl_call: failed to connect to node</samp>
							messages in the <span class="ph filepath">dbname.out</span> files, you might need to
							reduce the concurrency of the rebalancing process:
							<pre class="pre codeblock"><code>rebal run --concurrency 5 <var class="keyword varname">rebalance_plan_filename</var></code></pre>
This
							command reduces the number of concurrent shard moves to 5 from the
							default of 20.</li>
<li class="li p">It might be a temporary issue, such as an unreachable or
							overloaded node, leading to a timeout. Try running the rebalancing
							process again with the <span class="keyword cmdname">rebal run</span> command and the
							appropriate <var class="keyword varname">rebalance_plan_ filename</var>. Rebalancing in
							this way does not cause a shard to have too few copies, but it might
							result in shards with more than the expected number of copies.</li>
</ul>
If you checked the configuration and tried to rerun the shard moves, but
					the problem persists, contact support.</div>
</div></div>
<div class="section"><h2 class="title sectiontitle">Step 8: Verify the new node</h2>   
<div class="p">To confirm that the new node is set up correctly, log in to it with SSH and run
					<span class="keyword apiname">Weatherreport</span> with this
				command:<pre class="pre codeblock"><code>/opt/cloudant/bin/weatherreport</code></pre>
If errors are reported
				during the run, see the <a class="xref" href="clinstall_checking_health_cluster_with_weatherreport.html#clinstall_checking_health_cluster_with_weatherreport" title="Use these instructions to use the Weatherreport utility to check the health of your Cloudant cluster. Weatherreport is a command-line application that provides information about the status of a dbcore node or cluster. It is useful in troubleshooting cluster issues, such as increased latencies, low disk space, or node failures.">information</a> for
					<span class="keyword apiname">Weatherreport</span> checks, or call support.</div>

<p class="p">After the run finishes, check the rebalancing logs that are described in the <a class="xref" href="#clinstall_add_node__run_rebal">Run rebalancing </a> step
				for any errors.</p>

<div class="p">Verify whether the shards for different databases are now distributed across the recently added
				node. The following command checks the shard map for a database:
				<pre class="pre codeblock"><code>curl -X GET http://localhost:5986/dbs/default%2F<var class="keyword varname">&lt;database_name&gt;</var> | python -m json.tool | less</code></pre>
</div>


<p class="p">Look at the <samp class="ph codeph">by_node</samp> key in the output from this command to determine whether the
				shards are spread evenly across the cluster nodes.</p>
<div class="note note"><span class="notetitle">Note:</span> <span class="keyword apiname">Weatherreport</span> is
a command-line application that provides information about the status
of a <samp class="ph codeph">dbcore</samp> node or cluster. <span class="keyword apiname">Weatherreport</span> is
useful in troubleshooting cluster issues, such as increased latencies,
low disk space, or node failures. For more information about <span class="keyword apiname">Weatherreport</span>,
see <a class="xref" href="clinstall_checking_health_cluster_with_weatherreport.html#clinstall_checking_health_cluster_with_weatherreport" title="Use these instructions to use the Weatherreport utility to check the health of your Cloudant cluster. Weatherreport is a command-line application that provides information about the status of a dbcore node or cluster. It is useful in troubleshooting cluster issues, such as increased latencies, low disk space, or node failures.">Checking the health of a cluster with Weatherreport</a>.</div>
</div>
</div>
<div class="related-links">
<div class="familylinks">
<div class="parentlink"><strong>Parent topic:</strong> <a class="link" href="../topics/clinstall_maintenance_tasks_overview.html" title="Use the following tasks to maintain your Cloudant Local cluster.">Cloudant Local maintenance tasks</a></div>
</div>
</div>

<style>
.content pre.codeblock, .content pre.codeblock code {
  margin: 10px;
  float: none;
  background-color: inherit;
  color: inherit;
  position: relative;
  left: 0;
  top: 0;
  border-top-style: none; 
  border-top-width: 0; 
}

div.page-wrapper div.dark-box {

  width: 0px;
        
}  
</style>
