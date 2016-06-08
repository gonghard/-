1.由于需要下载相关jar包，先在项目的pom.xml里面配置相关的仓库地址
  	       
           <pluginRepositories>
		    	<pluginRepository>
				<id>sonatype-snapshots</id>
				<name>Sonatype Snapshots</name>
				<url>https://oss.sonatype.org/content/repositories/snapshots/</url>
				<releases>
					<enabled>false</enabled>
				</releases>
				<snapshots>
					<enabled>true</enabled>
				</snapshots>
			</pluginRepository>
	    	</pluginRepositories>
	    	<repositories>
			<repository>
				<id>sonatype-snapshots</id>
				<name>Sonatype Snapshots</name>
				<url>https://oss.sonatype.org/content/repositories/snapshots/</url>
				<releases>
					<enabled>false</enabled>
				</releases>
				<snapshots>
					<enabled>true</enabled>
				</snapshots>
			</repository>
	    	</repositories>
2.cargo插件配置
          
         <plugin>
				<groupId>org.codehaus.cargo</groupId>
				<artifactId>cargo-maven2-plugin</artifactId>
				<version>1.5.1-SNAPSHOT</version>
				<configuration>
					<container>
						<containerId>tomcat8x</containerId>
						<type>remote</type><!--远程部署-->
					</container>
					<configuration>
						<type>runtime</type>
						<properties>
							<cargo.hostname>192.168.0.174</cargo.hostname>
							<cargo.remote.uri>http://192.168.0.174:8080/manager/text/</cargo.remote.uri>
							<cargo.remote.username>admin</cargo.remote.username><!--请在tomcat下配置-->
							<cargo.remote.password>2013520</cargo.remote.password>
							<cargo.servlet.port>8080</cargo.servlet.port>
							<cargo.tomcat.ajp.port>8009</cargo.tomcat.ajp.port>
						</properties>
					</configuration>
					<deployables>
						<deployable>
							<type>war</type><!--以war包形式发布-->
							<properties>
								<context>/sosino-web</context>
							</properties>
						</deployable>
					</deployables>
				</configuration>
			</plugin>

3.由于本人用的是eclipse并且集成了所以直接执行以下命令，在命令窗口执行只需加上mvn

  cargo:redeploy 若在tomcat中容器中已经发布会先将其卸载重新发布，执行前请先确认tomcat-users.xml文件是否配置好相关用户权限可以参考4
 

4.tomcat用户权限配置

	  <role rolename="tomcat"/>  
	  <role rolename="manager"/>      
	  <role rolename="manager-script"/>  
	  <role rolename="manager-status"/>  
	  <role rolename="manager-jmx"/>      
	  <role rolename="manager-gui"/>  
	  <role rolename="admin"/>  
	  <role rolename="admin-gui"/>   
	  <user username="admin" password="2013520" roles="admin,admin-gui,manager,manager-gui,manager-script,manager-status,manager-jmx"/>  

5.注意cargo:redeploy执行第一次可以部署成功，但是部署第二次报如下错误

  	Failed to execute goal org.codehaus.cargo:cargo-maven2-plugin:1.5.1-SNAPSHOT:redeploy (default-cli) on project sosino-web: Execution default-cli of goal org.codehaus.cargo:cargo-maven2-plugin:1.5.1-SNAPSHOT:redeploy failed: Failed to undeploy [E:\sosino\workspace\sosino\sosino-web\target\sosino-web-0.0.1-SNAPSHOT.war]: The Tomcat Manager responded "FAIL - Unable to delete [E:\sosino\apache-tomcat-8.0.33\webapps\sosino-web]. The continued presence of this file may cause problems.


解决方案修改context.xml文件

  	<Context antiResourceLocking="true">表示不锁定