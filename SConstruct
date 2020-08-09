
from pathlib import Path
import glob
import os
import urllib.request


env = Environment(tools=['javac', 'jar'])
#env['JAVAC'] = env.Literal('C:/Program Files/Java/jdk1.8.0_131/bin/javac.exe')
#env['JAR'] = env.Literal('C:/Program Files/Java/jdk1.8.0_131/bin/jar.exe')
env['JAVAVERSION'] = '1.8'
env.Append(JAVACFLAGS=['-Xmaxerrs', '5', '-source', '1.8'])
dependency_map = {
    'javapoet-1.12.1.jar': 'https://oss.sonatype.org/service/local/repositories/releases/content/com/squareup/javapoet/1.12.1/javapoet-1.12.1.jar',
    'lz4-java-1.4.1.jar': 'https://repo1.maven.org/maven2/org/lz4/lz4-java/1.4.1/lz4-java-1.4.1.jar'
}

env.Prepend(_JAVACLASSPATH=['$_JARDEPS'])


def getJarDependency(env, target, source, for_signature):

    for dep in env.get('JAVACLASSPATH', []):
        if dep.startswith('depends'):
            os.makedirs('depends', exist_ok=True)
            if not os.path.exists(dep):
                urllib.request.urlretrieve(dependency_map[os.path.basename(dep)], dep)
        env.Depends(target[0], dep)
    return ''

env['_JARDEPS']=getJarDependency

env.Jar(
    target= 'build/arc-core.jar', 
    source=['repos/Arc/arc-core/src'],
    EXTRA_CLASSES={
        'repos/Arc/arc-core/src/arc/util/serialization/JsonValue.java':['JsonValue$1'],
        'repos/Arc/arc-core/src/arc/util/serialization/JsonWriter.java':['JsonWriter$1'],
        'repos/Arc/arc-core/src/arc/files/Fi.java':['Fi$2']}
)

def writeProcessors(env, target, source):
    
    with open(str(target[0]), 'w') as processor, open(str(target[1]), 'w') as plugin:
        for s in source:
            with open(str(s)) as f:

                contents = f.read()
                package_path = str(Path(str(s)).relative_to(
                    'repos/Mindustry/annotations/src/main/java')).replace(env['JAVASUFFIX'], ''
                    ).replace("/", "."
                    ).replace("\\", ".")

                if (' extends BaseProcessor' in contents
                    or (' extends AbstractProcessor' in contents 
                        and 'abstract class' not in contents)):
                    processor.write(package_path + '\n')

                elif ' implements Plugin' in contents:
                    plugin.write(package_path + '\n')

base_path = 'repos/Mindustry/annotations/src/main/resources/META-INF/services/'
processor = env.Command(
    target = [
        base_path + 'javax.annotation.processing.Processor',
        base_path + 'com.sun.source.util.Plugin',
    ],
    source = glob.glob('repos/Mindustry/annotations/src/main/java/**/*.java', recursive=True),
    action=writeProcessors
)

annotations_files = env.Jar(
    target='build/annotations.jar', 
    source=['repos/Mindustry/annotations/src/main/java'], 
    EXTRA_CLASSES={'repos/Mindustry/annotations/src/main/java/mindustry/annotations/impl/StructProcess.java':['StructProcess$1']},
    JAVACFLAGS=env['JAVACFLAGS'] + ['--add-exports', 'java.base/sun.reflect.annotation=ALL-UNNAMED',],
    JAVACLASSPATH=[
        'depends/javapoet-1.12.1.jar',
        'build/arc-core.jar',
        'C:/Program Files/Java/jdk1.8.0_131/lib/tools.jar',
        'repos/Mindustry/annotations/src/main/resources/META-INF/services'
    ])

env.Depends(annotations_files, processor)

for extenstion in ['g3d', 'box2d', 'fx', 'arcnet', 'freetype']:
    env.Jar(
        target= f'build/arc-{extenstion}.jar', 
        source=[f'repos/Arc/extensions/{extenstion}/src'],
        JAVACLASSPATH=[
            'build/arc-core.jar'])

env.Jar(
    target= 'build/rhino.jar', 
    source=['repos/rhino/src'])

flags = env['JAVACFLAGS'] + [
        '-s', 'repos/Mindustry/core/src', 
        '-processor', 'mindustry.annotations.entity.EntityProcess,mindustry.annotations.impl.AssetsProcess,mindustry.annotations.impl.CallSuperProcess,mindustry.annotations.impl.StructProcess,mindustry.annotations.misc.LoadRegionProcessor,mindustry.annotations.misc.LogicStatementProcessor,mindustry.annotations.remote.RemoteProcess'] + ['--add-exports', 'java.base/sun.reflect.annotation=ALL-UNNAMED',]

if os.path.exists('repos/Mindustry/core/src/mindustry/gen'):
    flags += ['-proc:none']

env.Jar(
    target='build/mindustry-core.jar', 
    source=['repos/Mindustry/core/src'],
    JAVACFLAGS=flags,
    JAVACLASSPATH=[
        'depends/javapoet-1.12.1.jar',
        'build/arc-fx.jar',
        'build/arc-g3d.jar',
        'build/arc-box2d.jar',
        'build/arc-arcnet.jar',
        'build/arc-freetype.jar',
        'build/arc-core.jar',
        'build/rhino.jar',
        'depends/lz4-java-1.4.1.jar',
        'build/annotations.jar',
    ])
