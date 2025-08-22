# Blender Python API 完全ドキュメント

## 目次
1. [概要](#概要)
2. [環境構築とセットアップ](#環境構築とセットアップ)
3. [主要モジュール詳細](#主要モジュール詳細)
4. [基本操作ガイド](#基本操作ガイド)
5. [高度なテクニック](#高度なテクニック)
6. [実践的なサンプルコード](#実践的なサンプルコード)
7. [トラブルシューティング](#トラブルシューティング)
8. [リファレンス](#リファレンス)

## 概要

Blender Python APIは、Blenderの全機能にプログラマティックにアクセスできる強力なインターフェースです。3Dモデリング、アニメーション、レンダリング、ゲーム開発など、あらゆる用途で使用できます。

### 主な特徴
- **包括性**: Blenderのほぼ全ての機能にアクセス可能
- **柔軟性**: スクリプトからアドオン開発まで幅広い用途
- **リアルタイム性**: 対話的な操作とスクリプト実行の両方をサポート
- **拡張性**: カスタムオペレータやパネルの作成が可能

---

## 環境構築とセットアップ

### Blender内でのPython実行

#### 1. テキストエディタでのスクリプト実行
```python
# Blender内のテキストエディタで実行
import bpy

# 基本的なオブジェクト作成
bpy.ops.mesh.primitive_cube_add(location=(0, 0, 0))
```

#### 2. Python コンソールでの対話実行
```python
# Blenderのpythonコンソールで対話的に実行
>>> import bpy
>>> bpy.context.scene.objects['Cube'].location.x = 2.0
```

#### 3. 外部スクリプトの実行
```bash
# コマンドラインからの実行
blender --background --python myscript.py
```

### 開発環境のセットアップ

#### Visual Studio Code用設定
```json
{
    "python.defaultInterpreterPath": "/Applications/Blender.app/Contents/Resources/4.0/python/bin/python3.11",
    "python.autoComplete.extraPaths": [
        "/Applications/Blender.app/Contents/Resources/4.0/scripts/modules"
    ]
}
```

#### オートコンプリート用スタブの生成
```python
# fake-bpy-module をインストール
pip install fake-bpy-module-latest
```

---

## 主要モジュール詳細

### 1. bpy モジュール

#### bpy.context - コンテキストアクセス
```python
import bpy

# アクティブオブジェクトの取得
active_obj = bpy.context.active_object

# 選択されたオブジェクトの取得
selected_objects = bpy.context.selected_objects

# 現在のシーン
current_scene = bpy.context.scene

# ビュー3Dエリア
view3d = bpy.context.space_data

# 編集モードかどうか
is_edit_mode = bpy.context.mode == 'EDIT_MESH'
```

#### bpy.data - データアクセス
```python
# 全てのオブジェクトにアクセス
for obj in bpy.data.objects:
    print(f"Object: {obj.name}, Type: {obj.type}")

# メッシュデータ
mesh_data = bpy.data.meshes["Cube"]

# マテリアル
materials = bpy.data.materials

# テクスチャ
textures = bpy.data.textures

# アクション（アニメーション）
actions = bpy.data.actions

# シーンデータ
scenes = bpy.data.scenes
```

#### bpy.ops - オペレータ
```python
# メッシュプリミティブの追加
bpy.ops.mesh.primitive_cube_add(size=2.0, location=(1, 2, 3))
bpy.ops.mesh.primitive_uv_sphere_add(radius=1.5)
bpy.ops.mesh.primitive_cylinder_add(vertices=8, radius=1.0, depth=2.0)

# オブジェクトの操作
bpy.ops.object.select_all(action='SELECT')
bpy.ops.object.delete(use_global=True)
bpy.ops.object.duplicate(linked=False)

# 変形操作
bpy.ops.transform.translate(value=(1, 0, 0))
bpy.ops.transform.rotate(value=1.5708, orient_axis='Z')
bpy.ops.transform.resize(value=(2, 2, 2))

# レンダリング
bpy.ops.render.render(animation=False, write_still=True)
bpy.ops.render.render(animation=True)
```

#### bpy.types - データ型とプロパティ
```python
# オブジェクトのプロパティにアクセス
obj = bpy.context.active_object

# 位置、回転、スケール
obj.location = (1.0, 2.0, 3.0)
obj.rotation_euler = (0, 0, 1.5708)  # radians
obj.scale = (2.0, 2.0, 2.0)

# 可視性
obj.hide_viewport = True
obj.hide_render = False

# レイヤー設定
obj.layers = [True] + [False] * 19
```

### 2. bmesh モジュール - 高度なメッシュ操作

#### 基本的なbmeshの使用
```python
import bmesh
import bpy

# アクティブオブジェクトのメッシュを取得
obj = bpy.context.active_object
mesh = obj.data

# bmeshインスタンスを作成
bm = bmesh.new()
bm.from_mesh(mesh)

# 頂点の追加
vert1 = bm.verts.new((1.0, 1.0, 0.0))
vert2 = bm.verts.new((-1.0, 1.0, 0.0))
vert3 = bm.verts.new((0.0, -1.0, 0.0))

# 面の作成
face = bm.faces.new([vert1, vert2, vert3])

# メッシュに変更を反映
bm.to_mesh(mesh)
bm.free()

# ビューポートの更新
bpy.context.view_layer.objects.active = obj
mesh.update()
```

#### bmeshでのメッシュ編集操作
```python
import bmesh
import bpy
from bmesh.ops import *

# 編集モードでbmeshを使用
def edit_mesh_with_bmesh():
    obj = bpy.context.edit_object
    mesh = obj.data
    bm = bmesh.from_edit_mesh(mesh)
    
    # 面の押し出し
    faces_to_extrude = [f for f in bm.faces if f.select]
    if faces_to_extrude:
        result = bmesh.ops.extrude_face_region(
            bm, 
            geom=faces_to_extrude
        )
        
        # 押し出された面を移動
        extruded_verts = [v for v in result['geom'] if isinstance(v, bmesh.types.BMVert)]
        bmesh.ops.translate(
            bm,
            vec=(0.0, 0.0, 1.0),
            verts=extruded_verts
        )
    
    # メッシュに変更を適用
    bmesh.update_edit_mesh(mesh)

# 使用例
if bpy.context.mode == 'EDIT_MESH':
    edit_mesh_with_bmesh()
```

### 3. mathutils モジュール - 数学計算

#### Vector と Matrix の基本操作
```python
from mathutils import Vector, Matrix, Euler
import math

# ベクトル操作
vec1 = Vector((1.0, 2.0, 3.0))
vec2 = Vector((4.0, 5.0, 6.0))

# ベクトル演算
dot_product = vec1.dot(vec2)
cross_product = vec1.cross(vec2)
magnitude = vec1.length
normalized = vec1.normalized()

# 行列操作
rotation_matrix = Matrix.Rotation(math.radians(45), 4, 'Z')
translation_matrix = Matrix.Translation((1, 2, 3))
scale_matrix = Matrix.Scale(2.0, 4)

# 複合変換
transform_matrix = translation_matrix @ rotation_matrix @ scale_matrix

# オイラー角
euler = Euler((math.radians(45), math.radians(30), 0), 'XYZ')
rotation_matrix = euler.to_matrix().to_4x4()
```

#### 実用的な座標変換
```python
from mathutils import Vector, Matrix

def world_to_local(obj, world_coord):
    """ワールド座標をオブジェクトのローカル座標に変換"""
    local_coord = obj.matrix_world.inverted() @ world_coord
    return local_coord

def local_to_world(obj, local_coord):
    """ローカル座標をワールド座標に変換"""
    world_coord = obj.matrix_world @ local_coord
    return world_coord

# 使用例
obj = bpy.context.active_object
world_pos = Vector((5.0, 3.0, 2.0))
local_pos = world_to_local(obj, world_pos)
```

### 4. bpy.utils モジュール - ユーティリティ関数

```python
import bpy.utils

# アドオンの登録/登録解除
bpy.utils.register_class(MyOperator)
bpy.utils.unregister_class(MyOperator)

# リソースパス関連
script_path = bpy.utils.script_path_user()
addon_path = bpy.utils.user_resource('SCRIPTS', path="addons")

# プレビュー
previews = bpy.utils.previews.new()
```

---

## 基本操作ガイド

### オブジェクトの作成と操作

#### 1. プリミティブオブジェクトの作成
```python
import bpy

# 立方体の作成
bpy.ops.mesh.primitive_cube_add(
    size=2.0,
    location=(0, 0, 0),
    rotation=(0, 0, 0)
)

# UV球の作成
bpy.ops.mesh.primitive_uv_sphere_add(
    radius=1.5,
    location=(3, 0, 0),
    segments=32,
    rings=16
)

# 円柱の作成
bpy.ops.mesh.primitive_cylinder_add(
    vertices=8,
    radius=1.0,
    depth=2.0,
    location=(-3, 0, 0)
)

# トーラスの作成
bpy.ops.mesh.primitive_torus_add(
    major_radius=2.0,
    minor_radius=0.5,
    location=(0, 3, 0)
)
```

#### 2. オブジェクトの選択と操作
```python
import bpy

# 全選択解除
bpy.ops.object.select_all(action='DESELECT')

# 名前でオブジェクトを選択
cube = bpy.data.objects.get("Cube")
if cube:
    cube.select_set(True)
    bpy.context.view_layer.objects.active = cube

# 選択されたオブジェクトの操作
for obj in bpy.context.selected_objects:
    obj.location.x += 1.0
    obj.scale = (1.5, 1.5, 1.5)
```

#### 3. オブジェクトの複製とインスタンス化
```python
import bpy

# 選択されたオブジェクトを複製
bpy.ops.object.duplicate(linked=False)

# リンク複製（メッシュデータを共有）
bpy.ops.object.duplicate(linked=True)

# 配列複製
def create_array(obj, count, offset):
    """オブジェクトを配列状に複製"""
    original_location = obj.location.copy()
    
    for i in range(1, count):
        # 複製
        bpy.ops.object.duplicate(linked=True)
        
        # 新しい位置に移動
        new_obj = bpy.context.active_object
        new_obj.location = original_location + (offset * i)

# 使用例
if bpy.context.active_object:
    create_array(bpy.context.active_object, 5, Vector((2, 0, 0)))
```

### メッシュの編集

#### 1. 頂点、エッジ、面の操作
```python
import bpy
import bmesh

def edit_mesh_components():
    # 編集モードに入る
    bpy.ops.object.mode_set(mode='EDIT')
    
    obj = bpy.context.edit_object
    mesh = obj.data
    bm = bmesh.from_edit_mesh(mesh)
    
    # 全ての面を選択
    for face in bm.faces:
        face.select = True
    
    # 面の押し出し
    bmesh.ops.extrude_face_region(bm, geom=bm.faces[:])
    
    # 最新の面を取得して移動
    latest_faces = [f for f in bm.faces if f.select]
    for face in latest_faces:
        face.translate(Vector((0, 0, 1)))
    
    # 変更を適用
    bmesh.update_edit_mesh(mesh)
    
    # オブジェクトモードに戻る
    bpy.ops.object.mode_set(mode='OBJECT')

# 実行
if bpy.context.active_object and bpy.context.active_object.type == 'MESH':
    edit_mesh_components()
```

#### 2. モディファイアの適用
```python
import bpy

def add_modifiers(obj):
    """オブジェクトに各種モディファイアを追加"""
    
    # サブサーフェスモディファイア
    subsurface = obj.modifiers.new(name="Subsurface", type='SUBSURF')
    subsurface.levels = 2
    
    # 配列モディファイア
    array = obj.modifiers.new(name="Array", type='ARRAY')
    array.count = 3
    array.relative_offset_displace[0] = 2.0
    
    # ソリッドファイモディファイア
    solidify = obj.modifiers.new(name="Solidify", type='SOLIDIFY')
    solidify.thickness = 0.1
    
    # ベベルモディファイア
    bevel = obj.modifiers.new(name="Bevel", type='BEVEL')
    bevel.width = 0.05
    bevel.segments = 3

# 使用例
if bpy.context.active_object:
    add_modifiers(bpy.context.active_object)
```

### マテリアルとテクスチャ

#### 1. マテリアルの作成と適用
```python
import bpy

def create_material(name, base_color=(1.0, 1.0, 1.0, 1.0)):
    """新しいマテリアルを作成"""
    material = bpy.data.materials.new(name=name)
    material.use_nodes = True
    
    # ノードにアクセス
    nodes = material.node_tree.nodes
    links = material.node_tree.links
    
    # デフォルトノードをクリア
    nodes.clear()
    
    # Principled BSDFノードを追加
    principled = nodes.new(type='ShaderNodeBsdfPrincipled')
    principled.inputs['Base Color'].default_value = base_color
    
    # Material Outputノードを追加
    output = nodes.new(type='ShaderNodeOutputMaterial')
    
    # ノードを接続
    links.new(principled.outputs['BSDF'], output.inputs['Surface'])
    
    return material

def apply_material_to_object(obj, material):
    """オブジェクトにマテリアルを適用"""
    if obj.data.materials:
        obj.data.materials[0] = material
    else:
        obj.data.materials.append(material)

# 使用例
red_material = create_material("Red_Material", (1.0, 0.0, 0.0, 1.0))
if bpy.context.active_object:
    apply_material_to_object(bpy.context.active_object, red_material)
```

#### 2. ノードベースマテリアルの構築
```python
import bpy

def create_complex_material(name):
    """複雑なノードマテリアルを作成"""
    material = bpy.data.materials.new(name=name)
    material.use_nodes = True
    
    nodes = material.node_tree.nodes
    links = material.node_tree.links
    
    # デフォルトノードをクリア
    nodes.clear()
    
    # ノードを作成
    principled = nodes.new(type='ShaderNodeBsdfPrincipled')
    output = nodes.new(type='ShaderNodeOutputMaterial')
    noise_texture = nodes.new(type='ShaderNodeTexNoise')
    color_ramp = nodes.new(type='ShaderNodeValToRGB')
    mapping = nodes.new(type='ShaderNodeMapping')
    coord = nodes.new(type='ShaderNodeTexCoord')
    
    # ノードの位置を設定
    principled.location = (300, 0)
    output.location = (600, 0)
    noise_texture.location = (-300, 0)
    color_ramp.location = (0, 0)
    mapping.location = (-600, 0)
    coord.location = (-900, 0)
    
    # ノードの設定
    noise_texture.inputs['Scale'].default_value = 5.0
    color_ramp.color_ramp.elements[0].color = (0.0, 0.0, 0.8, 1.0)
    color_ramp.color_ramp.elements[1].color = (0.8, 0.8, 1.0, 1.0)
    
    # ノードを接続
    links.new(coord.outputs['Generated'], mapping.inputs['Vector'])
    links.new(mapping.outputs['Vector'], noise_texture.inputs['Vector'])
    links.new(noise_texture.outputs['Fac'], color_ramp.inputs['Fac'])
    links.new(color_ramp.outputs['Color'], principled.inputs['Base Color'])
    links.new(principled.outputs['BSDF'], output.inputs['Surface'])
    
    return material

# 使用例
complex_mat = create_complex_material("Complex_Material")
```

---

## 高度なテクニック

### アニメーション

#### 1. キーフレームの作成と操作
```python
import bpy

def animate_object(obj, start_frame=1, end_frame=100):
    """オブジェクトをアニメーション"""
    
    # アニメーションデータを作成
    if not obj.animation_data:
        obj.animation_data_create()
    
    # 開始フレームの設定
    bpy.context.scene.frame_set(start_frame)
    obj.location = (0, 0, 0)
    obj.keyframe_insert(data_path="location")
    
    # 中間フレーム
    bpy.context.scene.frame_set(50)
    obj.location = (5, 5, 5)
    obj.keyframe_insert(data_path="location")
    
    # 終了フレーム
    bpy.context.scene.frame_set(end_frame)
    obj.location = (0, 0, 0)
    obj.keyframe_insert(data_path="location")
    
    # 補間方法を設定
    if obj.animation_data and obj.animation_data.action:
        for fcurve in obj.animation_data.action.fcurves:
            for keyframe in fcurve.keyframe_points:
                keyframe.interpolation = 'BEZIER'

# 使用例
if bpy.context.active_object:
    animate_object(bpy.context.active_object)
```

#### 2. カスタムプロパティのアニメーション
```python
import bpy

def create_custom_property_animation(obj):
    """カスタムプロパティをアニメーション"""
    
    # カスタムプロパティを追加
    obj["custom_value"] = 0.0
    
    # プロパティにキーフレームを追加
    bpy.context.scene.frame_set(1)
    obj["custom_value"] = 0.0
    obj.keyframe_insert(data_path='["custom_value"]')
    
    bpy.context.scene.frame_set(50)
    obj["custom_value"] = 10.0
    obj.keyframe_insert(data_path='["custom_value"]')
    
    bpy.context.scene.frame_set(100)
    obj["custom_value"] = 0.0
    obj.keyframe_insert(data_path='["custom_value"]')

# 使用例
if bpy.context.active_object:
    create_custom_property_animation(bpy.context.active_object)
```

### レンダリング

#### 1. レンダリング設定の制御
```python
import bpy
import os

def setup_render_settings():
    """レンダリング設定を構成"""
    scene = bpy.context.scene
    
    # 基本設定
    scene.render.engine = 'CYCLES'  # or 'BLENDER_EEVEE'
    scene.render.resolution_x = 1920
    scene.render.resolution_y = 1080
    scene.render.resolution_percentage = 100
    
    # Cyclesの設定
    scene.cycles.samples = 128
    scene.cycles.use_denoising = True
    
    # 出力設定
    scene.render.image_settings.file_format = 'PNG'
    scene.render.image_settings.color_mode = 'RGBA'
    scene.render.image_settings.compression = 15
    
    # 出力パス
    scene.render.filepath = "/tmp/render_"

def render_animation(start_frame=1, end_frame=250):
    """アニメーションをレンダリング"""
    scene = bpy.context.scene
    scene.frame_start = start_frame
    scene.frame_end = end_frame
    
    # 出力フォーマットを動画に設定
    scene.render.image_settings.file_format = 'FFMPEG'
    scene.render.ffmpeg.format = 'MPEG4'
    scene.render.ffmpeg.codec = 'H264'
    
    # レンダリング実行
    bpy.ops.render.render(animation=True)

# 使用例
setup_render_settings()
```

#### 2. ライティングの設定
```python
import bpy
from mathutils import Vector

def setup_three_point_lighting():
    """3点照明を設定"""
    
    # 既存のライトを削除
    bpy.ops.object.select_all(action='DESELECT')
    for obj in bpy.context.scene.objects:
        if obj.type == 'LIGHT':
            obj.select_set(True)
    bpy.ops.object.delete()
    
    # キーライト
    bpy.ops.object.light_add(type='SUN', location=(5, 5, 10))
    key_light = bpy.context.active_object
    key_light.data.energy = 3.0
    key_light.name = "Key_Light"
    
    # フィルライト
    bpy.ops.object.light_add(type='AREA', location=(-5, 5, 5))
    fill_light = bpy.context.active_object
    fill_light.data.energy = 1.0
    fill_light.data.size = 2.0
    fill_light.name = "Fill_Light"
    
    # バックライト
    bpy.ops.object.light_add(type='SPOT', location=(0, -5, 5))
    back_light = bpy.context.active_object
    back_light.data.energy = 2.0
    back_light.data.spot_size = 1.2
    back_light.name = "Back_Light"

# 使用例
setup_three_point_lighting()
```

### カメラワーク

#### 1. カメラの動的制御
```python
import bpy
from mathutils import Vector
import math

def create_camera_orbit_animation(target_object, radius=10, frames=100):
    """オブジェクトの周りを回るカメラアニメーション"""
    
    # カメラを作成
    bpy.ops.object.camera_add()
    camera = bpy.context.active_object
    
    target_location = target_object.location
    
    for frame in range(1, frames + 1):
        bpy.context.scene.frame_set(frame)
        
        # 円周上の位置を計算
        angle = (frame / frames) * 2 * math.pi
        x = target_location.x + radius * math.cos(angle)
        y = target_location.y + radius * math.sin(angle)
        z = target_location.z + 5
        
        camera.location = (x, y, z)
        
        # カメラをターゲットに向ける
        direction = target_location - camera.location
        camera.rotation_euler = direction.to_track_quat('-Z', 'Y').to_euler()
        
        # キーフレームを挿入
        camera.keyframe_insert(data_path="location")
        camera.keyframe_insert(data_path="rotation_euler")
    
    return camera

# 使用例
if bpy.context.active_object:
    create_camera_orbit_animation(bpy.context.active_object)
```

---

## 実践的なサンプルコード

### 1. プロシージャル建物生成
```python
import bpy
import bmesh
import random
from mathutils import Vector

def generate_building(width=5, depth=5, height=10, floors=3):
    """プロシージャルに建物を生成"""
    
    # 新しいメッシュを作成
    bm = bmesh.new()
    
    # 基本的な立方体を作成
    bmesh.ops.create_cube(bm, size=1.0)
    
    # スケールを適用
    bmesh.ops.scale(bm, vec=(width, depth, height), verts=bm.verts)
    
    # フロアの分割
    floor_height = height / floors
    
    for floor in range(1, floors):
        # 水平に分割するエッジを追加
        z_level = -height/2 + (floor * floor_height)
        
        # 分割線を作成
        bmesh.ops.bisect_plane(
            bm,
            geom=bm.verts[:] + bm.edges[:] + bm.faces[:],
            plane_co=(0, 0, z_level),
            plane_no=(0, 0, 1)
        )
    
    # ランダムに窓を作成
    for face in bm.faces:
        if abs(face.normal.z) < 0.1:  # 側面の面のみ
            if random.random() < 0.7:  # 70%の確率で窓を作成
                # インセット
                bmesh.ops.inset_individual(
                    bm, 
                    faces=[face], 
                    thickness=0.3
                )
                
                # 押し込み
                bmesh.ops.extrude_face_region(bm, geom=[face])
                bmesh.ops.translate(
                    bm,
                    vec=face.normal * -0.2,
                    verts=[v for v in bm.verts if v.select]
                )
    
    # 新しいオブジェクトを作成
    mesh = bpy.data.meshes.new("Building")
    bm.to_mesh(mesh)
    bm.free()
    
    obj = bpy.data.objects.new("Building", mesh)
    bpy.context.collection.objects.link(obj)
    
    return obj

# 使用例
building = generate_building(width=8, depth=6, height=15, floors=5)
```

### 2. パーティクルシステムの制御
```python
import bpy
import random

def create_particle_system(emitter_obj):
    """オブジェクトにパーティクルシステムを追加"""
    
    # パーティクルシステムを追加
    particle_system = emitter_obj.modifiers.new(
        name="ParticleSystem", 
        type='PARTICLE_SYSTEM'
    )
    
    settings = particle_system.particle_system.settings
    
    # パーティクルの基本設定
    settings.count = 1000
    settings.frame_start = 1
    settings.frame_end = 200
    settings.lifetime = 50
    
    # 物理設定
    settings.physics_type = 'NEWTON'
    settings.mass = 0.1
    settings.use_gravity = True
    settings.gravity_factor = 0.5
    
    # 速度設定
    settings.normal_factor = 5.0
    settings.factor_random = 2.0
    
    # レンダリング設定
    settings.render_type = 'HALO'
    settings.material_slot = 0
    
    return particle_system

def create_particle_emitter_material():
    """パーティクル用マテリアルを作成"""
    material = bpy.data.materials.new(name="Particle_Material")
    material.use_nodes = True
    
    # 発光設定
    nodes = material.node_tree.nodes
    principled = nodes.get("Principled BSDF")
    if principled:
        principled.inputs['Emission'].default_value = (1.0, 0.5, 0.0, 1.0)
        principled.inputs['Emission Strength'].default_value = 2.0
    
    return material

# 使用例
if bpy.context.active_object:
    particle_system = create_particle_system(bpy.context.active_object)
    particle_material = create_particle_emitter_material()
    bpy.context.active_object.data.materials.append(particle_material)
```

### 3. ジオメトリノードの活用
```python
import bpy

def create_geometry_nodes_modifier(obj, node_group_name="GeometryNodes"):
    """ジオメトリノードモディファイアを作成"""
    
    # ジオメトリノードモディファイアを追加
    modifier = obj.modifiers.new(name="GeometryNodes", type='NODES')
    
    # 新しいノードグループを作成
    node_group = bpy.data.node_groups.new(node_group_name, 'GeometryNodeTree')
    modifier.node_group = node_group
    
    # 入力と出力ノードを作成
    input_node = node_group.nodes.new('NodeGroupInput')
    output_node = node_group.nodes.new('NodeGroupOutput')
    
    # ソケットを作成
    node_group.interface.new_socket('Geometry', in_out='INPUT', socket_type='NodeSocketGeometry')
    node_group.interface.new_socket('Geometry', in_out='OUTPUT', socket_type='NodeSocketGeometry')
    
    # 基本的なノードを追加（例：散布）
    distribute_points = node_group.nodes.new('GeometryNodeDistributePointsOnFaces')
    instance_on_points = node_group.nodes.new('GeometryNodeInstanceOnPoints')
    
    # インスタンス用のプリミティブ
    cube_node = node_group.nodes.new('GeometryNodeMeshCube')
    
    # ノードを接続
    links = node_group.links
    links.new(input_node.outputs['Geometry'], distribute_points.inputs['Mesh'])
    links.new(distribute_points.outputs['Points'], instance_on_points.inputs['Points'])
    links.new(cube_node.outputs['Mesh'], instance_on_points.inputs['Instance'])
    links.new(instance_on_points.outputs['Instances'], output_node.inputs['Geometry'])
    
    # ノードの位置を調整
    input_node.location = (-400, 0)
    distribute_points.location = (-200, 0)
    cube_node.location = (-200, -200)
    instance_on_points.location = (0, 0)
    output_node.location = (200, 0)
    
    return modifier

# 使用例
if bpy.context.active_object:
    geo_modifier = create_geometry_nodes_modifier(bpy.context.active_object)
```

### 4. アドオン開発の基本構造
```python
import bpy
from bpy.types import Operator, Panel, PropertyGroup
from bpy.props import FloatProperty, IntProperty

bl_info = {
    "name": "Sample Addon",
    "author": "Your Name",
    "version": (1, 0),
    "blender": (4, 0, 0),
    "location": "View3D > Sidebar > Sample Tab",
    "description": "Sample addon for demonstration",
    "category": "Object",
}

class SAMPLE_OT_create_objects(Operator):
    """サンプルオペレータ：オブジェクトを作成"""
    bl_idname = "sample.create_objects"
    bl_label = "Create Sample Objects"
    bl_options = {'REGISTER', 'UNDO'}
    
    count: IntProperty(
        name="Count",
        description="Number of objects to create",
        default=5,
        min=1,
        max=100
    )
    
    spacing: FloatProperty(
        name="Spacing",
        description="Distance between objects",
        default=2.0,
        min=0.1,
        max=10.0
    )
    
    def execute(self, context):
        for i in range(self.count):
            bpy.ops.mesh.primitive_cube_add(
                location=(i * self.spacing, 0, 0)
            )
        return {'FINISHED'}

class SAMPLE_PT_panel(Panel):
    """サンプルパネル"""
    bl_label = "Sample Panel"
    bl_idname = "SAMPLE_PT_panel"
    bl_space_type = 'VIEW_3D'
    bl_region_type = 'UI'
    bl_category = "Sample"
    
    def draw(self, context):
        layout = self.layout
        
        col = layout.column()
        col.operator("sample.create_objects")

class SAMPLE_PG_properties(PropertyGroup):
    """サンプルプロパティグループ"""
    my_float: FloatProperty(
        name="My Float",
        description="A float property",
        default=1.0
    )

def register():
    bpy.utils.register_class(SAMPLE_OT_create_objects)
    bpy.utils.register_class(SAMPLE_PT_panel)
    bpy.utils.register_class(SAMPLE_PG_properties)
    bpy.types.Scene.sample_properties = bpy.props.PointerProperty(type=SAMPLE_PG_properties)

def unregister():
    bpy.utils.unregister_class(SAMPLE_OT_create_objects)
    bpy.utils.unregister_class(SAMPLE_PT_panel)
    bpy.utils.unregister_class(SAMPLE_PG_properties)
    del bpy.types.Scene.sample_properties

if __name__ == "__main__":
    register()
```

---

## トラブルシューティング

### よくある問題と解決法

#### 1. コンテキストエラー
```python
# ❌ 間違った方法
# bpy.ops.object.mode_set(mode='EDIT')  # コンテキストエラーが発生する可能性

# ✅ 正しい方法
if bpy.context.active_object and bpy.context.active_object.type == 'MESH':
    bpy.context.view_layer.objects.active = bpy.context.active_object
    bpy.ops.object.mode_set(mode='EDIT')
```

#### 2. メモリリークの防止
```python
import bmesh

def safe_bmesh_operation():
    """安全なbmesh操作"""
    bm = bmesh.new()
    try:
        # bmesh操作をここで実行
        bmesh.ops.create_cube(bm, size=1.0)
        
        # メッシュに適用
        mesh = bpy.data.meshes.new("NewMesh")
        bm.to_mesh(mesh)
        
    finally:
        # 必ずbmeshを解放
        bm.free()
    
    return mesh
```

#### 3. 例外処理
```python
def safe_object_operation(obj_name):
    """安全なオブジェクト操作"""
    try:
        obj = bpy.data.objects[obj_name]
        obj.location.x += 1.0
    except KeyError:
        print(f"Object '{obj_name}' not found")
        return False
    except Exception as e:
        print(f"Error operating on object: {e}")
        return False
    
    return True
```

### パフォーマンス最適化

#### 1. 大量オブジェクトの効率的な処理
```python
def batch_object_operations():
    """バッチ処理で効率化"""
    
    # ビューレイヤーの更新を一時停止
    bpy.context.view_layer.update()
    
    # バッチ処理
    for obj in bpy.data.objects:
        if obj.type == 'MESH':
            obj.location.z += 1.0
    
    # ビューポートの更新を強制
    bpy.context.view_layer.update()
```

#### 2. メッシュ操作の最適化
```python
def optimized_mesh_editing():
    """最適化されたメッシュ編集"""
    
    obj = bpy.context.active_object
    if not obj or obj.type != 'MESH':
        return
    
    # 編集モードに入る
    bpy.context.view_layer.objects.active = obj
    bpy.ops.object.mode_set(mode='EDIT')
    
    # bmeshを使用した効率的な編集
    bm = bmesh.from_edit_mesh(obj.data)
    
    # 複数の操作をまとめて実行
    bmesh.ops.subdivide_edges(bm, 
                             edges=bm.edges[:], 
                             cuts=2, 
                             use_grid_fill=True)
    
    # 一度だけメッシュを更新
    bmesh.update_edit_mesh(obj.data)
    
    bpy.ops.object.mode_set(mode='OBJECT')
```

---

## リファレンス

### 重要なデータ構造

#### bpy.context の主要属性
| 属性 | 説明 | 型 |
|------|------|-----|
| `active_object` | アクティブオブジェクト | `Object` |
| `selected_objects` | 選択されたオブジェクト | `list[Object]` |
| `scene` | 現在のシーン | `Scene` |
| `collection` | 現在のコレクション | `Collection` |
| `mode` | 現在のモード | `str` |
| `area` | 現在のエリア | `Area` |
| `space_data` | スペースデータ | `Space` |

#### bpy.data の主要コレクション
| コレクション | 説明 | 要素型 |
|-------------|------|--------|
| `objects` | 全オブジェクト | `Object` |
| `meshes` | 全メッシュデータ | `Mesh` |
| `materials` | 全マテリアル | `Material` |
| `textures` | 全テクスチャ | `Texture` |
| `images` | 全画像 | `Image` |
| `actions` | 全アクション | `Action` |
| `scenes` | 全シーン | `Scene` |

### 便利なユーティリティ関数

```python
import bpy
from mathutils import Vector, Matrix

def get_selected_vertices():
    """選択された頂点を取得"""
    if bpy.context.mode == 'EDIT_MESH':
        obj = bpy.context.edit_object
        bm = bmesh.from_edit_mesh(obj.data)
        return [v for v in bm.verts if v.select]
    return []

def safe_delete_object(obj_name):
    """安全にオブジェクトを削除"""
    if obj_name in bpy.data.objects:
        obj = bpy.data.objects[obj_name]
        bpy.data.objects.remove(obj, do_unlink=True)

def clear_scene():
    """シーンをクリア"""
    bpy.ops.object.select_all(action='SELECT')
    bpy.ops.object.delete(use_global=False)

def setup_basic_scene():
    """基本的なシーンセットアップ"""
    clear_scene()
    
    # ライト追加
    bpy.ops.object.light_add(type='SUN', location=(4, 4, 8))
    
    # カメラ追加
    bpy.ops.object.camera_add(location=(7, -7, 5))
    camera = bpy.context.active_object
    camera.rotation_euler = (1.1, 0, 0.785)
    
    # カメラをアクティブに設定
    bpy.context.scene.camera = camera
```

### 設定とプリファレンス

```python
# アドオンの有効化/無効化
bpy.ops.preferences.addon_enable(module="add_mesh_extra_objects")
bpy.ops.preferences.addon_disable(module="add_mesh_extra_objects")

# ユーザープリファレンス
prefs = bpy.context.preferences
prefs.view.show_splash = False
prefs.inputs.use_mouse_emulate_3_button = True

# シーン設定
scene = bpy.context.scene
scene.tool_settings.use_snap = True
scene.tool_settings.snap_elements = {'VERTEX'}
```

このドキュメントは、Blender Python APIの包括的なガイドとして作成されています。初心者から上級者まで、様々なレベルでの活用が可能です。

---

**注意**: このAPIドキュメントはBlender 4.0以降を対象としています。バージョンによって一部の機能や構文が異なる場合があります。