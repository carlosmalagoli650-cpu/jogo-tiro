function shoot() {
  if (!locked) return;

  raycaster.setFromCamera(
    new THREE.Vector2(0, 0),
    camera
  );

  raycaster.far = 2000;

  const targets = [];

  enemies.forEach(enemy => {
    enemy.traverse(obj => {
      if (obj.isMesh) targets.push(obj);
    });
  });

  const hits = raycaster.intersectObjects(targets, false);

  if (hits.length > 0) {
    const enemy = enemies.find(e =>
      e === hits[0].object ||
      e.getObjectById(hits[0].object.parent?.id)
    );

    if (enemy) damageEnemy(enemy);
  }
}
