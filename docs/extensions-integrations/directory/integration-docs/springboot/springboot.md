---
title: Getting Started
---

# Using Liquibase with Spring Boot


<p>The purpose of this tutorial is to guide you through the process of using Liquibase as part of your <a href="https://docs.spring.io/spring-boot/docs/current/reference/html/">Spring Boot</a> workflow. Spring Boot makes it easy to create standalone, production grade Spring-based applications.</p>
<h2>Uses</h2>
<p>Spring Boot allows you to create Java applications that can be started by using <code>java -jar</code> or <code>war</code> deployments.</p>
<p>With Spring Boot, some of the heavy lifting of configuring beans to set up things like messaging, database connection, migration, and others are already done for you. You only need to add the correct jar file on the classpath to be picked up by the framework for auto-configuration.</p>
<p>Some Spring Boot features include<span style="text-decoration: none;"> </span><span style="font-weight: bold;text-decoration: none;">Profiles</span><span style="text-decoration: none;">, </span><span style="font-weight: bold;text-decoration: none;">Logging</span><span style="text-decoration: none;">, </span><span style="font-weight: bold;text-decoration: none;">Security</span><span style="text-decoration: none;">, </span><span style="font-weight: bold;text-decoration: none;">Caching</span><span style="text-decoration: none;">, </span><span style="font-weight: bold;text-decoration: none;">Spring Integration</span><span style="text-decoration: none;">, </span><span style="font-weight: bold;text-decoration: none;">Testing</span><span style="text-decoration: none;">, </span>and more.</p>
<h2>Prerequisites</h2>
<p>Ensure you have Java Development Kit <a href="https://www.oracle.com/java/technologies/javase-downloads.html">(JDK 14+</a>)
    </p>
<h2>Using Liquibase with Spring Boot and the Gradle project</h2>
<p>The tutorial describes the use of Spring Boot with the Gradle project. To install Gradle and add it to your path, follow <a href="https://gradle.org/releases/">Gradle releases</a>.</p>
<p>Note: To use Spring Boot and Maven, see <a href="../using-springboot-with-maven">Using Liquibase with Spring Boot and Maven Project</a>.</p>
<p>To create the project, use <a href="https://start.spring.io">Spring Initializer</a>:</p>
<ol>
    <li>
        Under
<b>Project</b>, select <b>Gradle Project</b>. </li>
    <li>Select Java as your <b>Language</b>.</li>
    <li>Under <b>Spring Boot</b>, select 2.3.4.</li>
    <li>For <b>Packaging</b>, select <b>Jar</b>.</li>
    <li>Use version 11 for Java.</li>
</ol>
<p>After selecting your options, the project window needs to look similar to the screenshot:</p>
<p >
    <img src="../springboot.png" />
</p>
<p>Click <b>GENERATE</b> to download your project template as a <code>zip</code> file. Extract it and open in your IDE.</p>
<p>Note: Liquibase supports a variety of <a href="https://docs.liquibase.com/commands/home.html">commands</a>. For now, Spring Boot integrates the <code>update</code>, <code>future-rollback-sql</code>, <code>drop-all</code>, <code>update-testing-rollback</code>, and <code>clear-checksums</code> commands.</p>
<p>Spring Boot offers a subset of the Liquibase configuration options. In the table, the Spring Boot options are listed against the Liquibase options.</p>
<table>
    <thead>
        <tr>
            <th style="width: 35%">Spring Boot</th>
            <th style="width: 34%">Liquibase
            </th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><code>spring.liquibase.change-log</code>
            </td>
            <td><code>changelog-file</code>
            </td>
            <td>changelog configuration path</td>
        </tr>
        <tr>
            <td><code>spring.liquibase.labels</code>
            </td>
            <td><code>labels</code>
            </td>
            <td>Comma-separated list of runtime labels to use</td>
        </tr>
        <tr>
            <td><code>spring.liquibase.contexts</code>
            </td>
            <td><code>Contexts</code>
            </td>
            <td>Comma-separated list of runtime contexts to use</td>
        </tr>
        <tr>
            <td><code>spring.liquibase.database-change-log-lock-table</code>
            </td>
            <td><code>databaseChangeLogLockTableName</code>
            </td>
            <td>Name of table to use for tracking concurrent Liquibase usage</td>
        </tr>
        <tr>
            <td><code>spring.liquibase.database-change-log-table</code>
            </td>
            <td><code>databaseChangeLogTableName</code>
            </td>
            <td>Name of table to use for tracking change history</td>
        </tr>
        <tr>
            <td><code>spring.liquibase.default-schema</code>
            </td>
            <td><code>defaultSchemaName</code>
            </td>
            <td>Default database schema</td>
        </tr>
        <tr>
            <td><code>spring.liquibase.liquibase-schema</code>
            </td>
            <td><code>liquibaseSchemaName</code>
            </td>
            <td>Schema to use for Liquibase objects</td>
        </tr>
        <tr>
            <td><code>spring.liquibase.liquibase-tablespace</code>
            </td>
            <td><code>databaseChangeLogTablespaceName</code>
            </td>
            <td>Tablespace to use for Liquibase objects</td>
        </tr>
        <tr>
            <td><code>spring.liquibase.parameters.*</code>
            </td>
            <td><code>parameter.*</code>
            </td>
            <td>changelog parameters</td>
        </tr>
        <tr>
            <td><code>spring.liquibase.password</code>
            </td>
            <td><code>password</code>
            </td>
            <td>Login password of the database to migrate</td>
        </tr>
        <tr>
            <td><code>spring.liquibase.tag</code>
            </td>
            <td>--</td>
            <td>Tag name to use when applying database changes. Can also be used with <code>rollbackFile</code> to generate a rollback script for all existing changes associated with that tag</td>
        </tr>
        <tr>
            <td><code>spring.liquibase.url</code>
            </td>
            <td><code>url</code>
            </td>
            <td>JDBC URL of the database to migrate. If not set, the primary configured data source is used</td>
        </tr>
        <tr>
            <td><code>spring.liquibase.user</code>
            </td>
            <td><code>username</code>
            </td>
            <td>Login user of the database to migrate</td>
        </tr>
    </tbody>
</table>
<p>To start using Liquibase and Spring Boot with Gradle:</p>
<ol>
    <li>Open the existing Spring Boot <code>application.properties</code> file. To find the file, navigate to <code>src/main/resources/application.properties</code>.
    </li>
    <li>Add the following properties to run Liquibase migrations. Update the values depending on your database requirements:
    </li>
</ol><pre><code class="language-text">spring.datasource.url=jdbc:h2:~/liquibase;DB_CLOSE_ON_EXIT=FALSE
spring.datasource.username=sa
spring.datasource.password=
spring.liquibase.change-log=classpath:db/changelog/db.changelog-master.xml</code></pre>
<ol start="3">
    <li>Create a changelog file: <code>src/main/resources/db/changelog/db.changelog-master.xml</code>. Liquibase also supports the <code>.sql</code>, <code>.yaml</code>, or <code>.json</code> changelog formats.
For more information, see <a href="https://docs.liquibase.com/concepts/introduction-to-liquibase.htm">Introduction to Liquibase</a> and <a href="https://docs.liquibase.com/start/home.htm">Getting Started with Liquibase</a>.
    </li>
    <li>Add the following code snippet to your changelog file, including  changesets:
    </li>
</ol><pre xml:space="preserve"><code class="language-xml">&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;databaseChangeLog
xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:ext="http://www.liquibase.org/xml/ns/dbchangelog-ext"
xmlns:pro="http://www.liquibase.org/xml/ns/pro"
xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-latest.xsd
http://www.liquibase.org/xml/ns/dbchangelog-ext http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-ext.xsd
http://www.liquibase.org/xml/ns/pro http://www.liquibase.org/xml/ns/pro/liquibase-pro-latest.xsd"&gt;
&lt;changeSet id="202010211812" author="Julius Krah"&gt;
    &lt;createTable tableName="house"&gt;
        &lt;column name="id" type="bigint"&gt;
            &lt;constraints primaryKey="true" primaryKeyName="house_id_pk" /&gt;
        &lt;/column&gt;
        &lt;column name="owner" type="varchar(250)"&gt;
            &lt;constraints unique="true" uniqueConstraintName="house_owner_unq" /&gt;
        &lt;/column&gt;
        &lt;column name="fully_paid" type="boolean" defaultValueBoolean="false"&gt;&lt;/column&gt;
    &lt;/createTable&gt;
    &lt;createTable tableName="item"&gt;
        &lt;column name="id" type="bigint"&gt;
            &lt;constraints primaryKey="true" primaryKeyName="item_id_pk" /&gt;
        &lt;/column&gt;
        &lt;column name="name" type="varchar(250)" /&gt;
        &lt;column name="house_id" type="bigint"&gt;
            &lt;constraints nullable="false" notNullConstraintName="item_house_id_nn" /&gt;
        &lt;/column&gt;
    &lt;/createTable&gt;
    &lt;addAutoIncrement tableName="house" columnName="id" columnDataType="bigint" startWith="1" incrementBy="1" /&gt;
    &lt;addAutoIncrement tableName="item" columnName="id" columnDataType="bigint" startWith="1" incrementBy="1" /&gt;
    &lt;createSequence sequenceName="hibernate_sequence" incrementBy="1" startValue="1" /&gt;
    &lt;addForeignKeyConstraint baseTableName="item" baseColumnNames="house_id" constraintName="item_house_id_fk" referencedTableName="house" referencedColumnNames="id" /&gt;
&lt;/changeSet&gt;
&lt;/databaseChangeLog&gt;</code></pre>
<ol start="5">
    <li>Run your migration with the following command:
    </li>
</ol><pre xml:space="preserve"><code class="language-text">./gradlew bootRun</code></pre>
<p>Source code is available at <a href="https://github.com/juliuskrah/spring-boot-liquibase">https://github.com/juliuskrah/spring-boot-liquibase</a>.</p>
<p>Tip: If you did not use Spring Initializr, you might not have the <code>gradlew</code> script in your project. In this case, run the following command: <code>gradle bootRun</code>. Both <code>gradle</code> and <code>gradlew</code> create a Java <code>jar</code> for your application. The application <code>jar</code> is located in the <code>build/libs</code> directory of your Gradle project. You can execute the <code>jar</code> file by running <code>java -jar springbootProject-0.0.1-SNAPSHOT.jar</code>.</p>
<h2>Related links</h2>
<ul>
    <li><a href="https://juliuskrah.com/tutorial/2017/02/26/database-migration-with-liquibase-hikaricp-hibernate-and-jpa/" style="font-weight: bold;">Database Migration with Liquibase, HikariCP, Hibernate and JPA</a>
    </li>
    <li><a href="https://gradle.org/install/" style="font-weight: bold;">Gradle</a>
    </li>
</ul>