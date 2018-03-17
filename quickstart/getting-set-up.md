## 软件需求

Corda 使用的是行业标准的工具：

* Oracle JDK 8 JVM - 最低支持版本 8u131
* IntelliJ IDEA - 支持版本 2017.1, 2017.2 和 2017.3
* Git

我们也使用 Gradle 和 Kotlin，但是你不需要单独安装他们。一个独立的 Gradle wrapper 会被提供，而且它会下载正确版本的 Kotlin。

请注意：

* Corda 是在 JVM 中运行的。除了 Oracle JDK8，JVM 的实现是不支持的。但是如果你选择使用 OpenJDK，那么你需要安装 OpenJFX
* Corda 应用程序（CorDapp）可以使用任何目标是 JVM 的编程语言编写。但是 Corda 本身以及大部分的示例都是用 Kotlin 编写的。Kotlin 是官方的 Android 开发语言。如果你不熟悉 Kotlin，可以看看官方的[快速开始文档](https://kotlinlang.org/docs/tutorials/)，以及一些[相关的文章](https://kotlinlang.org/docs/tutorials/koans.html)。

使用这些推荐的软件开发 CorDapp可以最小化产生问题的几率，也能够让他人提供支持的时候更加容易。

## 安装指导 Set-up instruction

下边的指导内容可以告诉你如何配置一个 Corda 开发环境和如何运行一个基本的 CorDapp。如果你遇到任何问题，可以查看 [Troubleshooting 页面](https://docs.corda.net/troubleshooting.html)，或者在 Slack， [Stack Overflow](https://stackoverflow.com/questions/tagged/corda) 或 [论坛](https://discourse.corda.net/)中提问。

## Windows

### Java

1. 访问 [http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
2. 到 “Java SE Development Kit 8uXXX”（XXX 表示最新的版本号）
3. 选择 “Accept License Agreement”
4. 点击 jdk-8uXXX-windows-x64.exe 的下载链接（XXX 表示最新的版本号）
5. 下载并运行Java 的安装文件（使用默认设置）
6. 打开一个新的命令窗口然后运行 {% em %}highlighted !{% endem %} `java -version` 来测试一下 Java 是否正确安装了

```java
val timeWindow: TimeWindow? = tx.timeWindow

for ((inputs, outputs, _) in groups) {
    when (command.value) {
        is Commands.Move -> {
            val input = inputs.single()
            requireThat {
                "the transaction is signed by the owner of the CP" using (input.owner.owningKey in command.signers)
                "the state is propagated" using (outputs.size == 1)
                // Don't need to check anything else, as if outputs.size == 1 then the output is equal to
                // the input ignoring the owner field due to the grouping.
            }
        }

        is Commands.Redeem -> {
            // Redemption of the paper requires movement of on-ledger cash.
            val input = inputs.single()
            val received = tx.outputs.map { it.data }.sumCashBy(input.owner)
            val time = timeWindow?.fromTime ?: throw IllegalArgumentException("Redemptions must be timestamped")
            requireThat {
                "the paper must have matured" using (time >= input.maturityDate)
                "the received amount equals the face value" using (received == input.faceValue)
                "the paper must be destroyed" using outputs.isEmpty()
                "the transaction is signed by the owner of the CP" using (input.owner.owningKey in command.signers)
            }
        }

        is Commands.Issue -> {
            val output = outputs.single()
            val time = timeWindow?.untilTime ?: throw IllegalArgumentException("Issuances must be timestamped")
            requireThat {
                // Don't allow people to issue commercial paper under other entities identities.
                "output states are issued by a command signer" using (output.issuance.party.owningKey in command.signers)
                "output values sum to more than the inputs" using (output.faceValue.quantity > 0)
                "the maturity date is not in the past" using (time < output.maturityDate)
                // Don't allow an existing CP state to be replaced by this issuance.
                "can't reissue an existing state" using inputs.isEmpty()
            }
        }

        else -> throw IllegalArgumentException("Unrecognised command")
    }
}
```

### Git

1. 访问 [https://git-scm.com/download/win](https://git-scm.com/download/win)
2. 点击下载链接 “64-bit Git for Windows Setup”
3. 下载并运行 Git 安装文件（使用默认设置）
4. 打开一个新的命令窗口然后运行 git --version 来测试一下 Git 是否正确安装了

### IntelliJ

1. 访问 [https://www.jetbrains.com/idea/download/download-thanks.html?code=IIC](https://www.jetbrains.com/idea/download/download-thanks.html?code=IIC)
2. 下载并运行 InteliJ Community Edition 安装文件（使用默认设置）

### 下载示例项目

1. 打开一个命令窗口
2. 使用下边的命令克隆 CorDapp 的示例代码 git clone [https://github.com/corda/cordapp-example](https://github.com/corda/cordapp-example)
3. 通过运行 cd cordapp-example 命令进入 cordapp-example 文件夹下

### 从命令窗口中运行

1. 从 cordapp-example 文件夹下，运行 gradlew deployNodes 来部署节点
2. 运行 call kotlin-source/build/nodes/runnodes.bat 来开始运行节点
3. 等待所有的命令窗口中都显示“Webserver started up in XX.X sec”或者“Node for “NodeC” started up and registered in XX.XX sec”
4. 通过访问一下的前端页面来测试 CorDapp 是否成功运行起来了[http://localhost:10007/web/example/](http://localhost:10007/web/example/)

### 从 IntelliJ 运行

1. 打开 IntelliJ Community Edition
2. 在初始界面，点击“Open”（不要点击“Import Project”）并选择 cordapp-example 文件夹
3. 项目打开后，点击“File &gt; Project Structure”。在“Project SDK:”下，通过点击“New...”来设置项目的 SDK，然后选择 C:Program FilesJavajdk1.8.0\_XXX（XXX 是最新的版本号）。点击“OK”。
4. 点击“View &gt; Tool Windows &gt; Event Log”，然后点击“Import Gradle project”，然后点击“OK”。等待，在“Gradle Project Data To Import”窗口出现的时候，再次点击“OK”。
5. 等待 indexing 完成（在 IntelliJ 窗口的右下方会显示一个进度条，直到 indexing 结束）
6. 在屏幕的右上角，到绿色的“play”箭头的左侧，你应该会看到一个下拉列表。在里表中，选择“Run Example Cordapp - Kotlin”然后点击绿色的“play”箭头。
7. 等待至所有的命令窗口显示“Webserver started up in XX.X sec”
8. 通过访问一下的前端页面来测试 CorDapp 是否成功运行起来了 [http://localhost:10007/web/example/](http://localhost:10007/web/example/)

## Mac

### Java

1. 打开“System Preferences &gt; Java”
2. 在 Java 的控制面板中，如果有可用的更新的话，点击 “Update Now”
3. 在 “Software Update” 窗口，点击“Install Update”。如果需要，在弹出输入密码的提示窗的时候，输入你的密码并点击“Install Helper”。
4. 等待一个弹出的窗口提示你已经成功安装了更新，点击“Close”
5. 打开一个新的命令窗口然后运行 java -version 来测试一下 Java 是否正确安装了

### IntelliJ

1. 访问 [https://www.jetbrains.com/idea/download/download-thanks.html?code=IIC](https://www.jetbrains.com/idea/download/download-thanks.html?code=IIC)
2. 下载并运行 InteliJ Community Edition 安装文件（使用默认设置）

### 下载示例项目

1. 打开一个命令窗口
2. 使用下边的命令克隆 CorDapp 的示例代码 git clone [https://github.com/corda/cordapp-example](https://github.com/corda/cordapp-example)
3. 通过运行 cd cordapp-example 命令进入 cordapp-example 文件夹下

### 从命令窗口中运行

1. 从 cordapp-example 文件夹下，运行 gradlew deployNodes 来部署节点
2. 运行 call kotlin-source/build/nodes/runnodes.bat 来开始运行节点
3. 等待所有的命令窗口中都显示“Webserver started up in XX.X sec”或者“Node for “NodeC” started up and registered in XX.XX sec”
4. 通过访问一下的前端页面来测试 CorDapp 是否成功运行起来了 [http://localhost:10007/web/example/](http://localhost:10007/web/example/)

### 从 IntelliJ 运行

1. 打开 IntelliJ Community Edition
2. 在初始界面，点击“Open”（不要点击“Import Project”）并选择 cordapp-example 文件夹
3. 项目打开后，点击“File &gt; Project Structure”。在“Project SDK:”下，通过点击“New...”来设置项目的 SDK，然后选择 /Library/Java/JavaVirtualMachines/jdk1.8.0\_XXX（XXX 是最新的版本号）。点击“OK”。
4. 点击“View &gt; Tool Windows &gt; Event Log”，然后点击“Import Gradle project”，然后点击“OK”。等待，在“Gradle Project Data To Import”窗口出现的时候，再次点击“OK”。
5. 等待 indexing 完成（在 IntelliJ 窗口的右下方会显示一个进度条，直到 indexing 结束）
6. 在屏幕的右上角，到绿色的“play”箭头的左侧，你应该会看到一个下拉列表。在里表中，选择“Run Example Cordapp - Kotlin”然后点击绿色的“play”箭头。
7. 等待至所有的命令窗口显示“Webserver started up in XX.X sec”
8. 通过访问一下的前端页面来测试 CorDapp 是否成功运行起来了 [http://localhost:10007/web/example/](http://localhost:10007/web/example/)

## Corda 源代码

Corda 平台的源代码链接：[https://github.com/corda/corda.git](https://github.com/corda/corda.git)

Java 和 Kotlin 版本的 CorDapp 模板代码可以作为你的应用的基础代码

[https://github.com/corda/cordapp-template-java.git](https://github.com/corda/cordapp-template-java.git)

[https://github.com/corda/cordapp-template-kotlin.git](https://github.com/corda/cordapp-template-kotlin.git)

也可以通过查看其他的示例代码来研究更多的 Corda 基本概念

[https://www.corda.net/samples/](https://www.corda.net/samples/)

可以通过运行下边的命令将代码克隆到本地

git clone \[repo URL\]

