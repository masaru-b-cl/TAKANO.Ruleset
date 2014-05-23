TAKANO.Ruleset
==============

カスタマイズしたVisual Studioコード分析用のルールセットファイルをNuGetで配布します。

## 1. 使用するルールセットファイルを作成する

Visual Studioのコード分析サポート機能を使用し、必要なコード分析ルールを含むルールセットファイルを作成します（例：TAKANO.ruleset）。

## 2. MSBuildのpropsファイルを作成する

MSBuildに介入し、ビルド時に強制的に配布するルールセットファイルを参照させるようにします。そのために、MSBuildのpropsファイルを作成し、次のように配布したルールセットファイルを使うように指定します。

    <?xml version="1.0" encoding="utf-8" ?>
    <Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003" >
      <PropertyGroup>
        <CodeAnalysisRuleSet>$(SolutionDir)\packages\TAKANO.ruleset.1.0.0.0\ruleset\TAKANO.ruleset</CodeAnalysisRuleSet>
        <RunCodeAnalysis>true</RunCodeAnalysis>
      </PropertyGroup>
    </Project>

- CodeAnalysisRuleSet  
  配布するNuGetパッケージが展開されるフォルダーを指定します。このとき、配布するNuGetパッケージのバージョン番号を間違いなく指定するように注意します。
- RunCodeAnalysis  
  ビルド時に強制的にコード分析を行わせたいなら、この要素をtrueにします。あくまでルールセットの配布だけ行う場合は、要素自体を削除するか、falseを指定します。

## 3. nuspecファイルを作成する

作成したルールセットファイル、propsファイルを配布するためのnuspecファイルを作成します。ポイントはfile要素の指定方法です。

    <files>
      <file src="TAKANO.Ruleset.props" target="build" />
      <file src="TAKANO.ruleset" target="ruleset" />
    </files>

- ルールセットファイル  
  target属性に`ruleset`を指定することで、propsファイル内にも記載したとおり、NuGetパッケージ展開時にrulesetフォルダーに配置されるようになります。
- propsファイル  
  target属性に`build`を指定することで、csproj/vbprojなどのプロジェクトファイルに、propsファイルを参照するよう記述が追加されます。

## 4. NuGetパッケージを作成する

作成したnuspecファイルを使用して、`NuGet.exe pack TAKANO.Ruleset.nuspec`コマンドを実行してNuGetパッケージを作成すれば完了です。

なお、本コードではプロジェクトファイル（TAKANO.Ruleset.csproj）を書き換え、`NuGet.CommandLine`パッケージを利用して、ビルド時に自動的にNuGetパッケージを作成するようにしています。（参考：[kazuk/NuGetCustomBuilder](https://github.com/kazuk/NuGetCustomBuilder)）

      ...
      <Target Name="Package" AfterTargets="Build">
        <ItemGroup>
          <NuGetExe Include="$(SolutionDir)\packages\Nuget.CommandLine.*\**\NuGet.exe" />
        </ItemGroup>
        <Exec Command="&quot;@(NuGetExe)&quot; pack TAKANO.Ruleset.nuspec -OutputDirectory $(OutDir)" />
      </Target>
    </Project>


あとは、カスタムパッケージソースに指定したフォルダーやWebサイトに作成したNuGetパッケージを配置すれば、使用したいプロジェクトにてNuGetで`Install-Package TAKANO.Ruleset`するだけで、指定したルールセットでコード分析が行えます。